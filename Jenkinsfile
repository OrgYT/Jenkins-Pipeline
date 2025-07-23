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
                junit allowEmptyResults: true, testResults: 'reports/test-results.xml'
            }
        }

        stage('Code Coverage') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    sh 'npm run coverage'
                }
                // Note: This line seems incorrect; `junit` expects JUnit XML, not an HTML file
                // Consider removing or correcting:
                 junit allowEmptyResults: true, testResults: 'index.html'
            }
        }

        stage('SAST') {
            steps {
                sh 'echo $SONAR_SCANNER_HOME'

                withSonarQubeEnv(credentialsId: 'Sonar-servertoken') {
                    sh '''
                        $SONAR_SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.host.url=http://20.244.105.234:9000 \
                        -Dsonar.projectKey=Solar-system \
                        -Dsonar.javascript.lcov.reportPaths=./coverage/lcov.info \
                        -Dsonar.sources=app.js/
                    '''
                }

                waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-servertoken'
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
