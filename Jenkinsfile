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
            script {
                steps { 
                    def packageJSON = readJSON file: 'package.json'
                    appVersion = packageJSON.version
                    echo "${appVersion}"
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
