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
                        sh 'npm audit --audit-level=critical || true'
                    }
                }

                stage('OWASP Dependency Check') {
                    steps {
                        dependencyCheck additionalArguments: '''--scan ./ \
--out ./dependency-check-report \
--format ALL''', odcInstallation: 'owasp-10'
                    }
                }
            }
        }
    }
}
