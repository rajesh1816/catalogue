pipeline {
  agent {
    label 'AGENT-1'
  }

  environment {
    REGION_NAME = "us-east-1"
    ACC_ID = "887363634632"
    appVersion= ''
    COMPONENT = "catalogue"
  }

  options {
        timeout(time: 30, unit: 'MINUTES') 
        disableConcurrentBuilds()
    }

  stages {

    stage('Read App Version') {
      steps {
        script {
          def packageJSON = readJSON file: 'package.json'
          appVersion = packageJSON.version
          echo "Application version: ${appVersion}"
        }
      }
    }

    stage('Unit Testing') {
      steps {
        script {
          echo "Running unit tests..."
        }
      }
    }

    stage('Docker Build & Push to ECR') {
      steps {
        script {
            withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                sh """
                    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 887363634632.dkr.ecr.us-east-1.amazonaws.com

                    docker build -t ${ACC_ID}.dkr.ecr.${REGION_NAME}.amazonaws.com/$PROJECT/$COMPONENT:$"{appVersion}"

                    docker push ${ACC_ID}.dkr.ecr.${REGION_NAME}.amazonaws.com/$PROJECT/$COMPONENT:$"{appVersion}"
                """
             }
        }
      }
    }
  }

  post {
    success {
      echo "Pipeline completed successfully"
    }
    failure {
      echo "Pipeline failed"
    }
  }
}
