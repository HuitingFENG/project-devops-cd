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
    stage('Deploy Docker Image') {
        steps {
            script {
                sh "docker rm -f ${MY_GO_APP} || true"
                sh "docker run -d --name ${MY_GO_APP} -p 8081:8080 ${DOCKER_IMAGE}"
            }
        }
    }
  }
}