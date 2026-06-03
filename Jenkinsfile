pipeline {
    agent any

    environment {
        HOME = "${WORKSPACE}"
        NPM_CONFIG_CACHE = "${WORKSPACE}/.npm"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install & Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                    args '-u root'
                }
            }

            steps {
                sh '''
                    set -e

                    echo "Node version:"
                    node -v
                    npm -v

                    mkdir -p $NPM_CONFIG_CACHE

                    npm ci
                    npm test -- --watchAll=false
                '''
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                    args '-u root'
                }
            }

            steps {
                sh '''
                    set -e

                    mkdir -p $NPM_CONFIG_CACHE

                    npm ci
                    npm run build

                    ls -la build || true
                '''
            }
        }

        stage('E2E Tests (Playwright)') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.60.0-jammy'
                    reuseNode true
                    args '-u root'
                }
            }

            steps {
                sh '''
                    set -e

                    echo "Installing dependencies..."
                    npm ci

                    echo "Starting app..."
                    npx serve -s build -l 3000 &
                    npx wait-on http://localhost:3000

                    echo "Running Playwright tests..."
                    npx playwright test
                '''
            }
        }
    }

    post {
        always {
            echo "Publishing test results if available..."

            junit allowEmptyResults: true, testResults: 'jest-results/**/*.xml'

            archiveArtifacts artifacts: 'build/**', fingerprint: true, allowEmptyArchive: true
        }
    }
}
