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
        sh (script: "docker run --rm aquasec/trivy image --format json --output vuln-results.json chad38/jenkins-cicd:2023")
        
        
      }
    }
  }
  post {
     always {
       archiveArtifacts artifacts: '**/*.json', fingerprint: true, followSymlinks: false
       sh (script: 'docker-compose down')
       cleanWs deleteDirs: true, patterns: [[pattern: '*.zip, *.json', type: 'EXCLUDE']]
      }
    }
  }
