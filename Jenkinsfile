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
    stage("Docker Push") {
      steps {
        echo "Running in $WORKSPACE"
        dir("$WORKSPACE/azure-vote") {
          script {
            docker.withRegistry('', 'docker-hub') {
              def image = docker.build('chad38/jenkins-cicd:2023')
              image.push()
            }
          }
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
