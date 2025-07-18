pipeline {
    agent any

    tools {
        nodejs 'node-version'  // Make sure 'node-version' is defined in Jenkins Global Tools
    }

    environment {
        MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
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

                stage('Unit testing') {
                    steps {
                        withCredentials([usernamePassword(credentialsId: 'mongodb-credentials', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
                            sh 'npm test'
                        }
                        junit allowEmptyResults: true, testResults: 'test-results.xml'
                    }
                }

                stage('Code coverage') {
                    steps {
                        withCredentials([usernamePassword(credentialsId: 'mongodb-credentials', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
                            catchError(buildResult: 'SUCCESS', message: 'Coverage failed', stageResult: 'UNSTABLE') {
                                sh 'npm run coverage'
                            }
                        }

                        publishHTML([
                            allowMissing: false,
                            alwaysLinkToLastBuild: false,
                            icon: '',
                            keepAll: false,
                            reportDir: 'coverage/lcov-report',
                            reportFiles: 'index.html',
                            reportName: 'Coverage report',
                            reportTitles: 'Coverage',
                            useWrapperFileDirectly: true
                        ])
                    }
                }
            }
        }
    }
}
