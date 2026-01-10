pipeline {
    agent any 

    environment {
        PROJECT = 'roboshop'
        appVersion = ''
        REGION = 'us-east-1'
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
