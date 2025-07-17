pipeline{
    agent any

    tools{
        npm 'node-version'
    }

    stages{
        stage('npm dependencies'){
            steps{
                sh 'npm install --no audit'
            }
        }

    }
}
          
