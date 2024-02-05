pipeline {
  agent {
    label {
      label 'node1'
    }
  }
  stages {
    stage("Verify Branch") {
      steps {
        echo "$GIT_BRANCH"
      }
    }
    stage("Docker Build") {
      steps {
        sh(script: 'docker-compose build')
      }
    }
    stage("Start App") {
      steps {
        sh(script: 'docker-compose up -d')
      }
    }
    stage("Run Tests") {
      steps {
        sh(script: 'python3 -m pytest ./tests/test_sample.py')
      }
      post {
        success {
          echo "Tests Passed! :)"
        }
        failure {
          echo "Tests Failed :("
        }
      }
    }
    stage("Run Grype") {
      steps {
        grypeScan autoInstall: true, repName: 'grypeReport_${JOB_NAME}_${BUILD_NUMBER}.txt', scanDest: 'registry:chad38/jenkins-cicd:2023' 
      }
      post {
        always {
          recordIssues(
            tools: [grype()],
            aggregatingResults: true,
            )
        }
      }
    }
  }
  post {
     always {
       sh (script: 'docker-compose down')
      }
    }
  }
