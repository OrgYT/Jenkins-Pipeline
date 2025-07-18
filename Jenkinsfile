pipeline {
    agent any

    tools {
        nodejs 'node-version'
    }

    environment {
      MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
    }
    options {
      disableConcurrentBuilds() abortPrevious:true
    }

    stages {
        stage('Install NPM Dependencies') {
            steps {
                sh 'npm install --no-audit'
            }
        }

        stage('Security Scanning') {
            options {timestamps()}
            parallel {
                stage('NPM Audit - Critical Only') {
                    steps {
                        // Run npm audit with critical level, but don't fail pipeline on vulnerabilities
                        sh 'npm audit --audit-level=critical || true'
                    }
                }

                

                stage('Unit testing') {
                    steps {
                        withCredentials([usernamePassword(credentialsId: 'mongodb-credentials', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]){
                            sh 'npm test'
                        }
                        junit allowEmptyResults: true, stdioRetention:'', testResults: 'test-results.xml'
                    }
                }
            }
        }
    }
}
