pipeline {
    agent any

    tools{
        nodejs "nodejs-26.7.0"
    }

    environment {
        MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
        MONGO_DB_Creds = credentials('mongo-db-credentials')
        MONGO_DB_USER = credentials('mongo-db-username')
        MONGO_DB_PASSWORD = credentials('mongo-db-psw')
    }

    stages {
        stage('Installing Dependencies') {
            steps {
                sh 'npm install --no-audit'
            }
        }
        stage('Dependency Scanning') {
            parallel {
                stage('NPM Dependency Audit') {
                    steps {
                        sh '''
                           npm audit --audit-level=critical
                           echo $?
                        '''
                    }
                }
                stage('OWASP Dependency Check') {
                    steps {
                        dependencyCheck additionalArguments: '''
                            --scan './'
                            --out './'
                            --format 'ALL'
                            --prettyPrint
                        ''', odcInstallation: 'OWASP-DepCheck12'
                        dependencyCheckPublisher failedTotalCritical: 1, pattern: 'dependency-check-report.xml', stopBuild: true 
                    }
                }
            }
        }
        
        /*
        stage('Unit Tests'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'mongo-db-credentials', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
                // some block
                    sh 'npm test'
                }
                junit allowEmptyResults: true, keepProperties: true, testResults: 'test-results.xml'
            }
        }
        */

        stage('Code Coverage') {  
            steps {
                catchError(buildResult: 'SUCCESS', message: 'Oops! It will be fixed in future Releases', stageResult: 'UNSTABLE') {
                    sh 'npm run coverage'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'printenv'
                sh 'docker build -t abdul23/solar-system:$GIT_COMMIT .'
            }
        }

        stage('Trivy Vulnerability Scanner') {
            steps {
                sh '''
                    trivy image abdul23/solar-system:$GIT_COMMIT \
                    --severity LOW,MEDIUM \
                    --exit-code 0 \
                    --quiet \
                    --format json -o trivy-image-MEDIUM-results.json \
                    trivy image abdul23/solar-system:$GIT_COMMIT \
                    --severity HIGH,CRITICAL \
                    --exit-code 1 \
                    --quiet \
                    --format json -o trivy-image-CRITICAL-results.json
                '''
            }
            post {
                always {
                    sh '''
                    trivy convert \
                        --format template --template "@/usr/local/share/trivy/templates/html.tpl" \
                        --output trivy-image-MEDIUM-results.html trivy-image-MEDIUM-results.json

                    trivy convert \
                        --format template --template "@/usr/local/share/trivy/templates/html.tpl" \
                        --output trivy-image-CRITICAL-results.html trivy-image-CRITICAL-results.json

                    trivy convert \
                        --format template --template "@/usr/local/share/trivy/templates/junit.tpl" \
                        --output trivy-image-MEDIUM-results.xml trivy-image-MEDIUM-results.json

                    trivy convert \
                        --format template --template "@/usr/local/share/trivy/templates/junit.tpl" \
                        --output trivy-image-CRITICAL-results.xml trivy-image-CRITICAL-results.json
               '''
                }
            }
        }
    }
    post {
        always {
            // One or more steps need to be included within each condition's block.
            junit allowEmptyResults: true, keepProperties: true, testResults: 'dependency-check-junit.xml' 
            junit allowEmptyResults: true, keepProperties: true, testResults: 'trivy-image-MEDIUM-results.xml' 
            junit allowEmptyResults: true, keepProperties: true, testResults: 'trivy-image-CRITICAL-results.xml' 

            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: './',reportFiles: 'trivy-image-CRITICAL-results.html', reportName: 'Trivy Image Critical Vul Report',reportTitles: '', useWrapperFileDirectly: true])

            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: './',reportFiles: 'trivy-image-MEDIUM-results.html', reportName: 'Trivy Image Medium Vul Report',reportTitles: '', useWrapperFileDirectly: true]

            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true,
             reportDir: 'coverage/lcov-report/', reportFiles: 'index.html', reportName: 'Code Coverage HTML Report', reportTitles: '', useWrapperFileDirectly: true])           

            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'dependency-check-jenkins.html', reportName: 'Dependency Check HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}
