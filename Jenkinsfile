pipeline {
    agent any

    tools {
        nodejs 'node-version'  // Ensure this is configured in Jenkins > Global Tool Configuration
    }

    stages {
        stage('Install NPM Dependencies') {
            steps {
                sh 'npm install --no-audit'
            }
        }

        stage('Dependency Check') {
            steps {
                sh 'npm audit --audit-level=critical'
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '''--scan ./ \
--out ./ \
--format ALL''', odcInstallation: 'owasp-10'
            }
        }
    }
}
