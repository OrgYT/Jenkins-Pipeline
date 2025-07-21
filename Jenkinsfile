pipeline {
    agent any

    tools {
        nodejs 'node-version' // Ensure 'node-version' is defined in Jenkins Global Tools
    }

    environment {
        MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
        MONGO_CREDS = credentials('mongodb-credentials')
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
                        sh 'npm audit --audit-level=critical '
                    }
                }

                stage('Unit Testing') {
                    steps {
                        sh 'echo username - $MONGO_CREDS_USR'
                        sh 'npm test'
                        junit allowEmptyResults: true, testResults: 'test-results.xml'
                    }
                }

                stage('Code Coverage') {
                    steps {
                        sh 'npm coverage'

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
}
