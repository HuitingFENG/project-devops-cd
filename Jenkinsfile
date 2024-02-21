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
                sh 'kubectl apply -f k8sDev.yaml'
            }
        }
    }
    stage('Validate Development Deployment') {
        steps {
            script {
                def serviceUrl = sh(script: "minikube service my-go-app-service --url", returnStdout: true).trim()
                echo "Service URL is: ${serviceUrl}"
                sh 'curl -f ${serviceUrl}/whoami'
            }
        }
    }
    stage('Deploy to Production in Kubernetes') {
        when {
            expression { return currentBuild.result == null || currentBuild.result == 'SUCCESS' }
        }
        steps {
            script {
                sh 'kubectl apply -f k8sProd.yaml'
            }
        }
    }
  }
}