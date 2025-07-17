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
        stage('Dependency check'){
            steps{
                sh 'npm audit --auditlevel=critical'
            }
        }
        stage('OWASP check'){
            steps{
                dependencyCheck additionalArguments: 
             '''--scan./ \
                --out ./ \
                --format ALL \
                --prettyPrint
             ''', odcInstallation: 'owasp-10'

    }
}
    }
}
