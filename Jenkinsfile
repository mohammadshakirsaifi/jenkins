pipeline {
    agent any

    environment {
        NPM_CONFIG_CACHE = "/tmp/.npm"
    }

    stages {

        stage('Install & Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    mkdir -p /tmp/.npm

                    node --version
                    npm --version

                    # clean install (more stable than npm ci in CI flaky workspaces)
                    rm -rf node_modules package-lock.json
                    npm install

                    npm run build
                '''
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    npm test
                '''
            }
        }

        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    export NPM_CONFIG_CACHE=/tmp/.npm
                    mkdir -p /tmp/.npm

                    npm install -g serve

                    serve -s build -l 3000 &
                    sleep 10

                    npx playwright test --reporter=html
                '''
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true, testResults: 'jest-results/junit.xml'

            publishHTML([
                allowMissing: true,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'playwright-report',
                reportFiles: 'index.html',
                reportName: 'Playwright HTML Report'
            ])
        }
    }
}
