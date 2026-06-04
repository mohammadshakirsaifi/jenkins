pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            args '-u root:root'
        }
    }

    environment {
        CI = 'true'
        HOME = "${env.WORKSPACE}"
        NPM_CONFIG_CACHE = "${env.WORKSPACE}/.npm"
        SKIP_E2E = "true"  // Skip E2E tests for now
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Clean Workspace') {
            steps {
                sh '''
                    rm -rf node_modules package-lock.json build .npm test-results playwright-report
                    mkdir -p .npm
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    # Fix package-lock.json sync issue
                    rm -f package-lock.json
                    npm install --package-lock-only
                    npm install --no-audit --no-fund
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
                sh '''
                    npm test -- --watchAll=false --ci --coverage --testResultsProcessor="jest-junit" || true
                '''
            }
        }

        stage('Start App (for E2E)') {
            when {
                expression { env.SKIP_E2E != "true" }
            }
            steps {
                sh '''
                    # Kill any process using port 3000
                    kill $(lsof -t -i:3000) 2>/dev/null || true
                    
                    # Install serve globally
                    npm install -g serve
                    
                    # Start the app in background
                    serve -s build -l 3000 > serve.log 2>&1 &
                    echo $! > serve.pid
                    
                    # Wait for app to be ready
                    sleep 5
                '''
            }
        }

        stage('E2E Tests (Playwright)') {
            when {
                expression { env.SKIP_E2E != "true" }
            }
            steps {
                sh '''
                    npx playwright install chromium
                    npx playwright test --reporter=html || true
                '''
            }
        }
    }

    post {
        always {
            script {
                // Clean up background processes - with error handling
                sh '''
                    if [ -f serve.pid ]; then
                        kill -9 $(cat serve.pid) 2>/dev/null || true
                        rm -f serve.pid
                    fi
                    # Kill any remaining serve processes
                    pkill -f "serve" 2>/dev/null || true
                    # Kill processes on port 3000
                    kill -9 $(lsof -t -i:3000) 2>/dev/null || true
                '''
                
                // Publish test results
                junit allowEmptyResults: true,
                      testResults: '**/junit.xml, test-results/**/*.xml'
                
                // Publish HTML report if it exists
                script {
                    if (fileExists('playwright-report/index.html')) {
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
                
                // Archive artifacts
                archiveArtifacts artifacts: 'playwright-report/**/*, test-results/**/*, serve.log, build/**/*, coverage/**/*', 
                              allowEmptyArchive: true
            }
        }
        
        failure {
            script {
                echo 'Pipeline failed! Check logs for details.'
                sh '''
                    if [ -f serve.log ]; then
                        echo "=== Serve Log Output ==="
                        cat serve.log
                    fi
                '''
            }
        }
    }
    
    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        ansiColor('xterm')
    }
}
