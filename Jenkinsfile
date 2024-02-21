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
                sh 'kubectl run tmp-shell --rm -i --tty --image=curlimages/curl -- /bin/sh'
                sh 'curl http://my-go-app-service-dev:80/whoami'
                
                //def serviceUrl = sh(script: "minikube service my-go-app-service-dev --url", returnStdout: true).trim()
                // def expectedMessage = sh(script: "curl -s ${serviceUrl}/whoami", returnStdout: true).trim()
                // echo "Expected message is: ${expectedMessage}"
                // def pipelineExpectedMessage = '{"Name":"project-devops-cd","Title":"DevOps and Continous Deployment Team","State":"Efrei Paris"}'
                // if (expectedMessage == pipelineExpectedMessage) {
                //     echo "Messages match. Deployment validation successful."
                // } else {
                //     error "Messages do not match. Deployment validation failed."
                // }
                // def minikubeIp = '192.168.49.2' 
                // def nodePort = '30007'
                // def serviceUrl = "http://${minikubeIp}:${nodePort}/whoami"
                // echo "Service URL is: ${serviceUrl}"
                // sh 'curl -f ${serviceUrl}/whoami'
            }
        }
    }
    stage('Deploy to Production in Kubernetes') {
        when {
            expression { return currentBuild.result == null || currentBuild.result == 'SUCCESS' }
        }
        steps {
            script {
                sh 'open -a Terminal'
                sh 'kubectl apply -f K8sProd.yaml'
            }
        }
    }
  }
}