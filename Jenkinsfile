pipeline {
    agent any

    tools {
        nodejs 'node-version'  // Ensure 'node-version' is configured in Jenkins Global Tool Configuration
    }

    stages {
        stage('Install NPM Dependencies') {
            steps {
                sh 'npm install --no-audit'
            }
        }
    }
}
