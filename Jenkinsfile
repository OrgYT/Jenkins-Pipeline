pipeline {
    agent any

    tools {
        nodejs 'node-version' // Ensure 'node-version' is defined in Jenkins Global Tools
    }

    environment {
        MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
        MONGO_CREDS = credentials('mongodb-credentials')
        MONGO_USERNAME = credentials('MONGO_USER')
        MONGO_PASSWORD = credentials('MONGO-PASSWORD')
        SONARQUBE_CLI = tools 'sonarqube-7.1.0';
    }

    stages {
        stage('Install NPM Dependencies') {
            steps {
                sh 'npm install --no-audit'
            }
        }

        stage('Security Scanning & Testing') {
            options { timestamps() }

            parallel {
                stage('NPM Audit - Critical Only') {
                    steps {
                        // Prevent failure from stopping the pipeline
                        sh 'npm audit --audit-level=critical || true'
                    }
                }

                stage('Unit Testing') {
                    steps {
                        sh 'npm test'
                        junit allowEmptyResults: true, testResults: 'test-results.xml'
                    }
                }

                stage('Code Coverage') {
                    steps {
                        catchError(buildResult: 'SUCCESS', message: 'it has to be skipped', stageResult: 'UNSTABLE') {
                            sh 'npm run coverage'
                        }
                    }
                }
            }

              stage('SAST'){
                  steps{
                       $SONARQUBE_CLI/bin/sonar \
                          -Dsonar.host.url=http://20.244.105.234:9000 \
                          -Dsonar.token=sqp_aadda9e372a85262068e785c249acc0aa10e9c4f \
                          -Dsonar.projectKey=Solar-System-Project
                          -Dsonar.sources=.\

            post {
                always {
                    junit allowEmptyResults: true, testResults: 'test-results.xml'

                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        icon: '',
                        keepAll: false,
                        reportDir: 'coverage/lcov-report',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report',
                        reportTitles: 'Coverage',
                        useWrapperFileDirectly: true
                    ])
                }
            }
        }
    }
}
