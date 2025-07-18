pipeline {
    agent any

    tools {
        nodejs 'node-version'
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

        stage('Security Scanning') {
            parallel {
                stage('NPM Audit - Critical Only') {
                    steps {
                        // Run npm audit with critical level, but don't fail pipeline on vulnerabilities
                        sh 'npm audit --audit-level=critical || true'
                    }
                }

                stage('OWASP Dependency Check') {
                    steps {
                        // Run OWASP Dependency Check scan
                        dependencyCheck additionalArguments: '''
                            --scan ./ \
                            --out ./ \
                            --format ALL \
                            --prettyPrint
                        ''', odcInstallation: 'owasp-10'

                        

                        publishHTML([
                            allowMissing: true,
                            alwaysLinkToLastBuild: true,
                            icon: '',
                            keepAll: true,
                            reportDir: './',
                            reportFiles: 'dependency-check-jenkins.html',
                            reportName: 'Dependency HTML Report',
                            reportTitles: 'Report',
                            useWrapperFileDirectly: true
                        ])

                        junit allowEmptyResults: true, stdioRetention:'', testResults: 'dependency-check-junit.xml'
                    }
                }

                stage('Unit testing') {
                    steps {
                        withCredentials([usernamePassword(credentialsId: 'mongodb-creds', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
                            sh 'npm test'
                        }
                        junit allowEmptyResults: true, stdioRetention:'', testResults: 'test-results.xml'
                    }
                }
            }
        }
    }
}
