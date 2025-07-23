pipeline {
    agent any

    tools {
        nodejs 'node-version' // Ensure this is configured in Jenkins global tools
    }

    environment {
        MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
        MONGO_CREDS = credentials('mongodb-credentials')
        MONGO_USERNAME = credentials('MONGO_USER')
        MONGO_PASSWORD = credentials('MONGO-PASSWORD')
        SONAR_SCANNER_HOME = tool 'sonar-7.2' // Ensure this tool is also configured
    }

    stages {
        stage('Install Dependencies') {
            steps {
                sh 'npm install --no-audit'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'npm test'
                junit(
                    testResults: 'reports/test-results.xml',
                    allowEmptyResults: true
                )
            }
        }

        stage('Code Coverage') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    sh 'npm run coverage'
                }
                // You may add HTML coverage reporting here if needed
            }
        }

        stage('SAST') {
            steps {
                sh 'echo $SONAR_SCANNER_HOME'

                withSonarQubeEnv('sonarqube') {
                    sh '''
                        $SONAR_SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.host.url=http://20.244.105.234:9000 \
                        -Dsonar.projectKey=Solar-system \
                        -Dsonar.javascript.lcov.reportPaths=./coverage/lcov.info \
                        -Dsonar.sources=app.js/
                    '''
                }

                waitForQualityGate abortPipeline: true
            }
        }

        stage('Docker Build') {
            steps {
                sh 'printenv'
                sh 'docker build -t knightfury285/solar-system:$GIT_COMMIT .'
            }
        }
    }

    post {
        always {
            publishHTML([
                allowMissing: false,
                alwaysLinkToLastBuild: false,
                reportDir: 'coverage/lcov-report',
                reportFiles: 'index.html',
                reportName: 'Coverage Report',
                reportTitles: 'Coverage',
                keepAll: false
            ])
        }
    }
}
