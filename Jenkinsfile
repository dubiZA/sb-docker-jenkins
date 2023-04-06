pipeline {
  environment {
    registry = "dubiza/node"
    registryCredential = "dockerhub-dubiza"
    versionTag = "18-alpine3.17"
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
              sh "checkov -d . -s --skip-check CKV_DOCKER_2 --framework dockerfile -o cli -o junitxml --output-file-path console,checkov_results.xml"
              junit skipPublishingChecks: true, testResults: "checkov_results.xml"
            } catch (err) {
              junit skipPublishingChecks: true, testResults: "checkov_results.xml"
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

    stage("Trivy Image Scan") {
      steps {
        script {
          try {
            sh 'trivy image --no-progress --exit-code 0 --format template --template "@/usr/local/share/trivy/templates/junit.tpl" -o junit-report.xml --severity HIGH,CRITICAL $registry:$versionTag'
            junit skipPublishingChecks: true, testResults: "junit-report.xml"
          } catch (err) {
            junit skipPublishingChecks: true, testResults: "junit-report.xml"
            throw err
          }
        }
      }
    }

    stage("Push to Registry") {
      steps {
        script {
          docker.withRegistry("https://index.docker.io/v1/", registryCredential) {
            dockerImage.push()
          }
        }
      }
    }
  }
}