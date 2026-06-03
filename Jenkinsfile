pipeline {
    agent any

    stages {
/*
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm ci
                    npm run build
                '''
            }
        }
*/
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm ci
                    npm test -- --watchAll=false
                '''
            }
        }

        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.60.0-jammy'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    npm ci
                    npx serve -s build -l 3000 &
                    npx wait-on http://localhost:3000
                    npx playwright test
                '''
            }
        }
    }

    post {
        always {
            junit 'jest-results/**/*.xml'
        }
    }
}
