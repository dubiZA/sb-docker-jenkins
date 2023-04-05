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
        stash includes: "**/Dockerfile", name: "dockerfile"
      }
    }

    stage("Checkov Dockerfile Scan") {
      steps {
        script {
          docker.image("bridgecrew/checkov:latest").inside("--entrypoint=''") {
            unstash "dockerfile"
            try {
              sh "checkov -d . -s --framework dockerfile -o cli -o junitxml --output-file-path console,results.xml"
              junit skipPublishingChecks: true, testResults: "results.xml"
            } catch (err) {
              junit skipPublishingChecks: true, testResults: "results.xml"
              throw err
            }
          }
        }
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
          docker.withRegistry(registry, registryCredential) {
            dockerImage.push()
          }
        }
      }
    }
  }

  options {
    preserveStashes()
    timestamps()
  }
}