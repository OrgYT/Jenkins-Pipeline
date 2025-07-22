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
        SONAR_SCANNER_HOME = tool 'sonar-7.2'
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
                        sh 'npm audit --audit-level=critical'
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
                    }
                }

                stage('Code Coverage') {
                    steps {
                        catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                            sh 'npm run coverage'
                        }
                    }
                }

                stage('SAST') {
                    steps {
                        sh 'echo $SONAR_SCANNER_HOME'
                        sh ''' 
                            $SONAR_SCANNER_HOME/bin/sonar \
                              -Dsonar.host.url=http://20.244.105.234:9000 \
                              -Dsonar.token=sqp_d8bd61f98cb3d7e6a978e2b0cc853d25964876d7 \
                              -Dsonar.projectKey=Solar-system \
                              -Dsonar.source=./
                        '''
                    }
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
