pipeline {
  environment {
    registry = "dubiza/alpine"
    registryCredential = "dockerhub-dubiza"
    versionTag = "3.17.0"
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
          dockerImage = docker.build(registry + ":${versionTag}")
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
        sh "docker image rm $registry:$versionTag"
      }
    }
  }
}