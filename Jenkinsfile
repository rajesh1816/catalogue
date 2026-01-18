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
    /* parameters {
        booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')

    } */

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

        /* stage('Sonar Scan') {
            environment {
                SCANNER_HOME = tool 'sonar-8.0'
            }
            steps {
                withSonarQubeEnv('sonar-8.0') {
                    sh "${SCANNER_HOME}/bin/sonar-scanner"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') { // 
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('Dependabot Alerts Check') {
            environment {
                GITHUB_TOKEN = credentials('git-token')
            }
            steps {
                script {
                    // GitHub repo details
                    def owner = "rajesh1816"
                    def repo = "catalogue"

                    // Call GitHub REST API
                    def response = sh(
                        script: """curl -s -H "Accept: application/vnd.github+json" \
                                        -H "Authorization: token ${env.GITHUB_TOKEN}" \
                                        https://api.github.com/repos/${owner}/${repo}/dependabot/alerts""",
                        returnStdout: true
                    ).trim()

                    // Parse JSON
                    def alerts = readJSON text: response

                    // Filter open alerts with high or critical severity
                    def criticalAlerts = alerts.findAll { alert ->
                        alert.state == 'open' && (alert.security_advisory.severity == 'critical' || alert.security_advisory.severity == 'high')
                    }

                    // Fail the pipeline if any critical/high alerts exist
                    if (criticalAlerts.size() > 0) {
                        echo "❌ Found ${criticalAlerts.size()} high/critical Dependabot alerts!"
                        criticalAlerts.each { alert ->
                            echo "Package: ${alert.dependency.package.name} | Severity: ${alert.security_advisory.severity} | Path: ${alert.dependency.manifest_path}"
                        }
                        error("Pipeline aborted due to critical/high Dependabot alerts.")
                    } else {
                        echo "✅ No critical/high Dependabot alerts found. Safe to proceed."
                    }
                }
            }
        } */



        stage('Build & Push to ECR') {
            steps {
                script {
                    withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                        sh """
                            # Login to ECR
                            aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com

                            # Build single-arch Docker image (amd64)
                            docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} .
                            
                            # Push image to ECR
                            docker push ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                            
                        """
                    }
                }
            }
        }

        stage('Update Image in CD Repo') {
            steps {
                script {
                    def owner = "rajesh1816"
                    def repo  = "eks-argocd"

                    sh """
                    rm -rf ${repo}
                    git clone https://github.com/${owner}/${repo}.git
                    cd ${repo}

                    # Update image version
                    sed -i "s/imageVersion:.*/imageVersion: ${appVersion}/" values-dev.yaml

                    git add values-dev.yaml
                    git commit -m "Update catalogue image to ${appVersion}"
                    git push
                    """
                }
            }
}



        /* stage('Trigger Deploy') {
            when{
                expression { params.deploy }
            }
            steps {
                script {
                    build job: 'catalogue-cd',
                    parameters: [
                        string(name: 'appVersion', value: "${appVersion}"),
                        string(name: 'deploy_to', value: 'dev')
                    ],
                    propagate: false,  
                    wait: false 
                }
            }
        } */
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