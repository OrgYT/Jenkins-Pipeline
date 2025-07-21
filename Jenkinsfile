pipeline {
    agent any

    tools {
        nodejs 'node-version' // Make sure this tool is configured in Jenkins global tools
    }

    environment {
        MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
        MONGO_CREDS = credentials('mongodb-credentials')
        MONGO_USERNAME = credentials('MONGO_USER')
        MONGO_PASSWORD = credentials('MONGO-PASSWORD')
        SONAR_CLI = tools('sonarqube-7.1.0') // Secure SonarQube token
    }

    stages {
        stage('Install Dependencies') {
            steps {
                sh 'npm install --no-audit'
            }
        }

        stage('Testing & Coverage') {
            parallel {
                stage('NPM Audit (Critical Only)') {
                    steps {
                        sh 'npm audit --audit-level=critical || true'
                    }
                }

                stage('Unit Tests') {
                    steps {
                        catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                            sh '''
                                mkdir -p reports
                                mocha app-test.js \
                                  --timeout 10000 \
                                  --reporter mocha-junit-reporter \
                                  --reporter-options mochaFile=reports/test-results.xml \
                                  --exit
                            '''
                        }
                        junit allowEmptyResults: true, testResults: 'reports/test-results.xml'
                    }
                }

                stage('Code Coverage') {
                    steps {
                        catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                            sh 'npm run coverage'
                        }
                    }
                }
            }
        }

        stage('SAST (SonarQube)') {
            steps {
                withSonarQubeEnv('MySonarQubeServer') { // Make sure this is configured in Jenkins
                    sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=Solar-System-Project \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=http://20.244.105.234:9000 \
                          -Dsonar.login=sqp_aadda9e372a85262068e785c249acc0aa10e9c4f
                    '''
                }
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true, testResults: 'reports/test-results.xml'

            publishHTML([
                allowMissing: false,
                alwaysLinkToLastBuild: false,
                reportDir: 'coverage/lcov-report',
                reportFiles: 'index.html',
                reportName: 'Coverage Report',
                reportTitles: 'Coverage',
                keepAll: false
            ])
        }
    }
}
