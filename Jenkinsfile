pipeline {
    agent {
        label 'AGENT-1'
    }

    environment {
        PROJECT = 'roboshop'
        appVersion = ''
        REGION = 'us-east-1'
        COMPONENT = "catalogue"
    }

    // build
    stages { 
        stage('Read app version') {
            steps {
                script { 
                    def packageJSON = readJSON file: 'package.json'
                    appVersion = packageJSON.version
                    echo "${appVersion}"
                }
            }
        }

        stage('install dependencies') {
            steps {
                script { 
                    sh '''
                        npm install
                    '''
                }
            }
        }

        stage('unit testing') {
            steps {
                script { 
                    sh '''
                        echo "running unit test cases"
                    '''
                }
            }
        }

        stage('Build & Push to ECR') {
            steps {
                script {
                    withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                        sh """
                            aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 887363634632.dkr.ecr.us-east-1.amazonaws.com

                            docker build -t 887363634632.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} .

                            docker push 887363634632.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                        """
                    }
                }
            }
        }

    }



    post { 
        always {
            echo 'I will always say Hello again!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
