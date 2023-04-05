pipeline {
  agent any
  stages {
    stage('Checkout Git Repo') {
      parallel {
        stage('Checkout Git Repo') {
          steps {
            git(url: 'https://github.com/dubiZA/sb-docker-jenkins', branch: 'main')
          }
        }

        stage('WhoAmI') {
          steps {
            sh 'whoami'
          }
        }

      }
    }

    stage('Build Image') {
      steps {
        sh 'docker image build -t dubiza/hellonode .'
      }
    }

  }
}