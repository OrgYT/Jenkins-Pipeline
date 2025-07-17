pipeline {
    agent any

    tools {
        nodejs 'node-version'  // Replace with your configured Node.js tool name
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
                        // Prevent failure from stopping pipeline if vulnerabilities are found
                        sh 'npm audit --audit-level=critical'
                    }
                }

                stage('OWASP Dependency Check') {
                    steps {
                        dependencyCheck additionalArguments: '''\
--scan ./ \
--out ./ \
--format ALL \
--prettyPrint
''', odcInstallation: 'owasp-10'

                        dependencyCheckPublisher failedTotalCritical: 1, pattern: 'dependency-check-junit.xml'
                        publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'dependency-check-jenkins.html', reportName: 'Dependency HTML Report', reportTitles: 'Report', useWrapperFileDirectly: true])
                        junit allowEmptyResults: true, testResults: 'dependency-check-junit.xml'
                    }
                }
            }
        }
    }
}
