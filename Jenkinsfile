pipeline {
  environment {
    registry = "dubiza/sb-docker-jenkins"
    registryCredential = "dockerhub-dubiza"
    dockerImage = ""
  }
  agent any

  stages {
    stage("Github Checkout") {
      steps {
        git branch: "main", url: "https://github.com/dubiZA/sb-docker-jenkins.git"
      }
    }

    stage("Build Image") {
      steps {
        script {
          dockerImage = docker.build(registry + ":${BUILD_NUMBER}")
        }
      }
    }

    stage("Push to Dockerhub") {
      steps {
        script {
          docker.withRegistry("https://registry.hub.docker.com", registryCredential) {
            dockerImage.push()
          }
        }
      }
    }

    stage("Cleanup") {
      steps {
        sh "docker image rm $registry:$BUILD_NUMBER"
      }
    }
  }
}