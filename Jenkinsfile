pipeline {
  environment {
    registry = "dubiza/node"
    registryCredential = "dockerhub-dubiza"
    versionTag = "18-alpine3.17.0"
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

    stage("Trivy Operations") {
      parallel {
        stage("Scan Image") {
          steps {
            script {
              try {
                sh "trivy image --no-progress --exit-code 0 --format template --template '@/usr/local/share/trivy/templates/junit.tpl' -o trivy_report.xml --severity HIGH,CRITICAL $registry:$versionTag"
                junit skipPublishingChecks: true, testResults: "trivy_report.xml"
              } catch (err) {
                junit skipPublishingChecks: true, testResults: "trivy_report.xml"
                throw err
              }
            }
          }
        }

        stage("Generate SBOM") {
          steps {
            script {
              sh "trivy image --format cyclonedx --output sbom_cyclonedx.json $registry:$versionTag"
              archiveArtifacts artifacts: "**/sbom_cyclonedx.json", onlyIfSuccessful: true
            }
          }
        }
      }
    }

    stage("Upload Artifacts") {
      parallel {
        stage("Push Image to Registry") {
          steps {
            script {
              docker.withRegistry("https://index.docker.io/v1/", registryCredential) {
                dockerImage.push()
              }
            }
          }
        }
        stage("Upload SBOM to GitHub") {
          steps {
            sh "echo 'Uploading SBOM to GH'"
          }
        }
      }
    }

    stage("Cleanup") {
      steps {
        sh "docker image rm -f $registry:$versionTag"
      }
    }
  }
}