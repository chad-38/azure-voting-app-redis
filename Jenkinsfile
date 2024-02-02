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
    stage("Run Clair") {
      agent {label 'node1'}
      steps {
        sh(script: 'docker run -p 5432:5432 -d --name db arminc/clair-db:latest')
        sh(script: 'docker run -p 6060:6060 --link db:postgres -d --name clair arminc/clair-local-scan:latest')
      }
    }

    stage("Run Clair Scan") {
      agent {label 'node1'}
      steps {
        sh(script: '/home/ubuntu/go/bin/clair-scanner --ip=127.0.0.1 chad38/jenkins-cicd:2023')
        
      }
    } 
  }
  post {
     always {
       sh (script: 'docker-compose down')
      }
    }
  }