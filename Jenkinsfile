pipeline {
  agent any

  environment {
      USER_NAME = "Eric",
      DC_WEBHOOK = credentials('dc-webhook')
  }

  parameters {
    choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Select the environment to deploy to')
  }

  stages {
    stage('First Stage') {
      steps {
        echo "Hello, ${USER_NAME}!"
        echo "Selected environment: ${params.ENVIRONMENT}"
      }

      when {
        allOf {
          branch 'main',
          environment name: 'ENVIRONMENT', value: 'prod'
        }
        return {
          echo "This stage will only run on the main branch and when the environment is set to prod."
        }
      }
    }

    stage('Parallel Stage') {
      agent {
        label 'for-parallel-steps'
      }

      failFast true

      parallel {
            stage('Lint') {
            steps {
                echo "Running linting..."
                sh 'npm run lint'
              }
            }

            stage('Test') {
            steps {
                echo "Running tests..."
                sh 'npm run test'
              }
            }
      }
    }

    stage ('Build Stage') {
      steps {
        retry(3) {
          echo "Building the application..."
          sh 'npm run build'
        }

        timeout(time: 10, unit: 'SECONDS') {
          echo "This build step will timeout after 10 seconds."
          sh 'sleep 15'
        }
      }
    }
  }

  post {
    always {
      echo "Running post actions..."
    }

    success {
      echo "Pipeline completed successfully!"
      sh 'curl -X POST -H "Content-Type: application/json" -d \'{"text":"Pipeline completed successfully!"}\' $DC_WEBHOOK'
    }

    failure {
      echo "Pipeline failed!"
      sh 'curl -X POST -H "Content-Type: application/json" -d \'{"text":"Pipeline failed!"}\' $DC_WEBHOOK'
    }
  }

}