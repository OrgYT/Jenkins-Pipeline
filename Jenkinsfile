pipeline {
    agent any

    tools {
        nodejs 'node-version' 
    }

    stages {
        stage('Show Node & NPM Version') {
            steps {
                sh '''
                    node -v
                    npm -v
                '''
            }
        }
    }
}

          
