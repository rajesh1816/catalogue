pipeline {
    agent {
        label 'AGENT-1'
    }

    environment {
        PROJECT   = 'roboshop'
        COMPONENT = 'catalogue'
        REGION    = 'us-east-1'
        ACC_ID    = '887363634632'
        appVersion = ''
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    stages {
        // =============================
        stage('Read app version') {
            steps {
                script {
                    def packageJSON = readJSON file: 'package.json'
                    appVersion = packageJSON.version
                    echo "App version: ${appVersion}"
                }
            }
        }

        // =============================
        stage('Install dependencies') {
            steps {
                sh 'npm install'
            }
        }

        // =============================
        stage('Unit testing') {
            steps {
                sh 'echo "Running unit tests..."'
            }
        }

        // =============================
        /* stage('SonarQube Scan') {
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
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        
        // =============================
        stage('Dependabot Alerts Check') {
            environment {
                GITHUB_TOKEN = credentials('git-token')
            }
            steps {
                script {
                    def owner = "rajesh1816"
                    def repo = COMPONENT
                    def response = sh(
                        script: """curl -s -H "Accept: application/vnd.github+json" \
                                    -H "Authorization: token ${env.GITHUB_TOKEN}" \
                                    https://api.github.com/repos/${owner}/${repo}/dependabot/alerts""",
                        returnStdout: true
                    ).trim()

                    def alerts = readJSON text: response
                    def criticalAlerts = alerts.findAll { alert ->
                        alert.state == 'open' && (alert.security_advisory.severity == 'critical' || alert.security_advisory.severity == 'high')
                    }

                    if (criticalAlerts.size() > 0) {
                        echo "❌ Found ${criticalAlerts.size()} high/critical Dependabot alerts!"
                        criticalAlerts.each { alert ->
                            echo "Package: ${alert.dependency.package.name} | Severity: ${alert.security_advisory.severity} | Path: ${alert.dependency.manifest_path}"
                        }
                        error("Pipeline aborted due to critical/high Dependabot alerts.")
                    } else {
                        echo "✅ No critical/high Dependabot alerts found."
                    }
                }
            }
        } */
        


        // =============================
        stage('Build & Push Docker image') {
            steps {
                script {
                    withAWS(credentials: 'aws-creds', region: REGION) {
                        sh """
                            # Login to ECR
                            aws ecr get-login-password --region ${REGION} | \
                                docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com

                            # Build Docker image
                            docker build -t ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} .

                            # Push image to ECR
                            docker push ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                        """
                    }
                }
            }
        }

        // =============================
        stage('Update Image in CD Repo') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'git-token', variable: 'GIT_TOKEN')]) {
                        sh 'rm -rf eks-argocd'

                        // Clone CD repo securely
                        sh "git clone https://$GIT_TOKEN@github.com/rajesh1816/eks-argocd.git"

                        // Go into catalogue folder inside the repo
                        dir('eks-argocd/catalogue') {
                            // Update image version
                            sh "sed -i 's|imageVersion:.*|imageVersion: ${appVersion}|' values-dev.yaml"

                            // Git commit & push
                            sh """
                                git config user.email "ci@company.com"
                                git config user.name "jenkins-ci"

                                git add values-dev.yaml
                                git commit -m "Update catalogue image to ${appVersion}" || echo "No changes to commit"
                                git push https://$GIT_TOKEN@github.com/rajesh1816/eks-argocd.git
                            """
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished!'
            deleteDir()  // Clean workspace
        }
        success {
            echo '✅ Pipeline succeeded!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
