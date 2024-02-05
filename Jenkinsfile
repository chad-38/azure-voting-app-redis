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
        sh(script: 'docker pull chad38/jenkins-cicd:2023')
      }
    }

    stage('Container Scanning') {
      parallel {
        stage("Run Clair Scan") {
          agent {label 'node1'}
          steps {
            sh(script: 'wget -qO clair-scanner https://github.com/arminc/clair-scanner/releases/download/v8/clair-scanner_linux_amd64')
            sh(script: 'chmod +x clair-scanner')
            sh(script: 'mv clair-scanner /tmp')
            sh(script: '/tmp/clair-scanner --ip=localhost chad38/jenkins-cicd:2023')
            sleep time: 1, unit: 'MINUTES'
           }
        }
        //stage("Run Grype") {
        // agent {label 'node1'}
        //  steps {
        //    grypeScan autoInstall: false, repName: 'grypeReport_${JOB_NAME}_${BUILD_NUMBER}.txt', scanDest: 'registry:chad38/jenkins-cicd:2023'
        //  }
        //}
      }
    }

    stage("QA Deploy") {
      agent {label 'kube-control'}
      when {
        branch 'feature/k8-deploy'
      }
      steps {
        sh(script: 'kubectl create ns QA')
        sh(script: 'kubectl apply -f azure-vote-all-in-one-redis.yaml -n QA')
      }
    }
    stage('Approve Deploy to Prod') {
      agent {label 'kube-control'}
      when {
        branch 'feature/k8-deploy'
      }
      options {
        timeout(time: 1, unit: 'HOURS')
      }
      steps {
        input message: "Deploy to PROD?"
      }
    }
    
    stage('PROD Deploy') {
      agent {label 'kube-control'}
      when {
        branch 'feature/k8-deploy'
      }
      steps {
        sh(script: 'kubectl create ns PROD')
        sh(script: 'kubectl apply -f azure-vote-all-in-one-redis.yaml -n PROD')
      }
    }

  }
  post {
     always {
       sh (script: 'docker-compose down')
       sh (script: 'docker container rm clair db -f')
      }
    }
  }