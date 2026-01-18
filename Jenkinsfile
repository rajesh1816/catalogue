pipeline {
    agent {
        label 'AGENT-1'
    }

    environment {
        PROJECT = 'roboshop'
        appVersion = ''
        REGION = 'us-east-1'
        COMPONENT = "catalogue"
        ACC_ID = "887363634632"
    }

    options {
        timeout(time: 30, unit: 'MINUTES') 
        disableConcurrentBuilds()
    }
    parameters {
        booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')

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

        stage('Sonar Scan') {
            environment {
                SCANNER_HOME = tool 'sonar-8.0'
            }
            steps {
                withSonarQubeEnv('sonar-8.0') {
                    sh "${SCANNER_HOME}/bin/sonar-scanner"
                }
            }
        }



        stage('Build & Push to ECR') {
            steps {
                script {
                    withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                        sh """
                            aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com

                            docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} .

                            docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
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
