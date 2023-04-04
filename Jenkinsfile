pipeline {
  agent any
  stages {
    stage('Checkout Git Repo') {
      steps {
        git(url: 'https://github.com/dubiZA/sb-docker-jenkins', branch: 'main')
      }
    }

  }
}