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
    stage("Trivy Scan") {
      steps {
        sh(script: 'docker run --rm aquasec/trivy ')
        sh (script: 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v ${WORKSPACE}/trivy-cache:/root/.cache/ aquasec/trivy --exit-code 0 --severity LOW,MEDIUM chad38/jenkins-cicd:2023')
        sh (script: 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v ${WORKSPACE}/trivy-cache:/root/.cache/ aquasec/trivy --exit-code 1 --severity HIGH,CRITICAL chad38/jenkins-cicd:2023')
      }
    }
  }
  post {
     always {
       sh (script: 'docker-compose down')
      }
    }
  }
