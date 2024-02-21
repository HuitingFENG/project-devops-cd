def dockerImage = ''


pipeline {

  agent {
    label 'jenkins-slave'
  }
  
  environment {
    DOCKER_HOST = 'unix:///var/run/docker.sock'
    DOCKER_IMAGE = 'huitingfeng/project-devops-cd:latest'
    MY_GO_APP = 'my-go-app'
  }

  stages {
    stage('Cloning Git') {
      steps {
        git branch: 'main', credentialsId: 'githubID', url: 'https://github.com/HuitingFENG/project-devops-cd.git'
      }
    }
    stage('Building Docker image') {
      steps {
        script {
          dir('webapi') {
            dockerImage = docker.build("${DOCKER_IMAGE}")
          }
        }
      }
    }
    stage('Publish Docker Image') {
      steps {
        script {
          withDockerRegistry(credentialsId: 'dockerID') {
            dockerImage.push()
          }

        }
      }
    }
    stage('Load Image to Minikube') {
        steps {
            script {
                sh 'eval $(minikube -p minikube docker-env)'
                sh "minikube image load ${DOCKER_IMAGE}"
            }
        }
    }
    stage('Deploy Docker Image') {
        steps {
            script {
                sh "docker rm -f ${MY_GO_APP} || true"
                sh "docker run -d --name ${MY_GO_APP} -p 8081:8080 ${DOCKER_IMAGE}"
            }
        }
    }
    stage('Deploy to Development in Kubernetes') {
        steps {
            script {
                sh 'minikube addons enable ingress'
                sh 'kubectl apply -f K8sDev.yaml'
            }
        }
    }
    stage('Check deployed dev service in Kubernetes') {
        steps {
            script {
                sh 'kubectl get pods,deployments,svc'
                
            }
        }
    }
    stage('Validate Development Deployment') {
        steps {
            script {
                echo "test"
                // def podStatuses = sh(script: "kubectl get pods -l app=my-go-app,environment=development -o jsonpath='{.items[*].status.phase}'", returnStdout: true).trim()
                // def statuses = podStatuses.split()
                // def allRunning = statuses.every { status -> status == 'Running' }

                // if (allRunning) {
                //     echo "All pods in the development deployment are running."
                // } else {
                //     error "One or more pods in the development deployment are not running."
                // }
            }
        }
    }
    stage('Deploy to Production in Kubernetes') {
        when {
            expression { return currentBuild.result == null || currentBuild.result == 'SUCCESS' }
        }
        steps {
            script {
                sh 'kubectl apply -f K8sProd.yaml'
            }
        }
    }
  }
}