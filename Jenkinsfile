pipeline {
  agent any

  environment {
    USER_NAME = "Eric"
  }

  parameters {
    choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Select the environment to deploy to')
  }

  stages {
    stage('First Stage') {
      steps {
        echo "Hello, ${USER_NAME}!"
        echo "Selected environment: ${params.ENVIRONMENT}"
        echo "This stage will only run on the main branch and when the environment is set to prod."
      }

      when {
        allOf {
          branch 'main'
          expression { return params.ENVIRONMENT == 'prod' }
        }
      }
    }

    stage('Install Stage') {
      tools {
        nodejs 'node-20'
      }
      steps {
        sh 'node -v'
        sh 'npm -v'
        sh 'npm ci'
        stash name: 'node-modules', includes: 'node_modules/**'
      }
    }

    stage('Parallel Stage') {
      failFast true
      parallel {
          stage('Lint') {
            agent any
            steps { 
              unstash 'node-modules'
              sh 'npm run lint' 
            }
          }
          stage('Test') {
            agent any
            steps { 
              unstash 'node-modules'
              sh 'npm run test' 
              }
          }
      }
    }

    stage('Build Stage Input') {
      agent none
      input {
        message "Do you want to proceed with the build?"
      }

      steps {
        echo "User confirmed to proceed with the build."
      }
      
      when {
        expression { return params.ENVIRONMENT == 'prod'}
      }
    }

    stage ('Build Stage') {
      steps {
        echo "Building the application..."
        unstash 'node-modules'
        retry(3) {
          sh 'npm run build'
        }
      }
    }
  }

  post {
  success {
    withCredentials([string(credentialsId: 'dc-webhook', variable: 'DC_WEBHOOK')]) {
      sh '''
        curl -X POST -H "Content-Type: application/json" \
             -d "{\\"text\\":\\"Pipeline completed successfully!\\"}" \
             "$DC_WEBHOOK"
      '''
    }
  }

  failure {
    withCredentials([string(credentialsId: 'dc-webhook', variable: 'DC_WEBHOOK')]) {
      sh '''
        curl -X POST -H "Content-Type: application/json" \
             -d "{\\"text\\":\\"Pipeline failed!\\"}" \
             "$DC_WEBHOOK"
      '''
    }
  }
  }
}