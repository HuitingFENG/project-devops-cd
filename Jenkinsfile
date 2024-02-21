def dockerImage = ''


pipeline {

  agent {
    label 'jenkins-slave'
  }
  
  environment {
    DOCKER_HOST = 'unix:///var/run/docker.sock'
  }

  stages {
    stage('Cloning Git') {
      steps {
        git branch: 'main', credentialsId: 'githubID', url: 'https://github.com/HuitingFENG/project-devops-cd.git'
      }
    }
    stage('Building docker image') {
      steps {
        script {
          dir('webapi') {
            dockerImage = docker.build("huitingfeng/project-devops-cd:latest")
          }
        }
      }
    }
    stage('Publish docker Image') {
      steps {
        script {
          withDockerRegistry(credentialsId: 'dockerID') {
            dockerImage.push()
          }

        }
      }
    }
  }
}