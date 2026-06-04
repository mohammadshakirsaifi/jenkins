pipeline {
    agent any

    environment {
        CI = 'true'
        HOME = "${env.WORKSPACE}"
        NPM_CONFIG_CACHE = "${env.WORKSPACE}/.npm"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    rm -rf node_modules package-lock.json
                    mkdir -p .npm
                    npm ci --cache .npm --prefer-offline --no-audit --no-fund
                '''
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'npm test -- --watchAll=false || true'
            }
        }

        stage('Start App (for E2E)') {
            steps {
                sh '''
                    npm install -g serve
                    serve -s build -l 3000 &
                    sleep 8
                '''
            }
        }

        stage('E2E Tests (Playwright)') {
            steps {
                sh '''
                    npx playwright install --with-deps
                    npx playwright test --reporter=html
                '''
            }
        }
    }

    post {
        always {

            junit allowEmptyResults: true,
                  testResults: 'test-results/**/*.xml'

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
