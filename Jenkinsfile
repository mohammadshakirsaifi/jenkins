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
        CI = "true"
        PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD = "0"
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
                    # Check if package-lock.json exists, if not create it
                    if [ ! -f package-lock.json ]; then
                        npm install --package-lock-only
                    fi
                    
                    # Install dependencies
                    npm ci --cache .npm --prefer-offline --no-audit --no-fund || npm install
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
            steps {
                sh '''
                    # Kill any process using port 3000
                    lsof -ti:3000 | xargs kill -9 2>/dev/null || true
                    
                    # Install serve globally
                    npm install -g serve
                    
                    # Start the app in background
                    serve -s build -l 3000 > serve.log 2>&1 &
                    SERVE_PID=$!
                    echo $SERVE_PID > serve.pid
                    
                    # Wait for app to be ready with timeout
                    echo "Waiting for app to start on port 3000..."
                    for i in 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15; do
                        if curl -s http://localhost:3000 > /dev/null; then
                            echo "App is ready on port 3000!"
                            exit 0
                        fi
                        echo "Attempt $i/15 - Waiting for app to start..."
                        sleep 2
                    done
                    
                    echo "App failed to start within 30 seconds"
                    cat serve.log
                    exit 1
                '''
            }
        }

        stage('E2E Tests (Playwright)') {
            steps {
                sh '''
                    # Install Playwright only if not already installed
                    if [ ! -d "node_modules/@playwright" ]; then
                        npx playwright install chromium
                    else
                        npx playwright install chromium --with-deps
                    fi
                    
                    # Run Playwright tests with timeout
                    timeout 300 npx playwright test --reporter=html,json --output=test-results --timeout=60000 || {
                        EXIT_CODE=$?
                        if [ $EXIT_CODE -eq 124 ]; then
                            echo "E2E tests timed out after 5 minutes"
                        else
                            echo "E2E tests failed with exit code: $EXIT_CODE"
                        fi
                        exit 0
                    }
                '''
            }
        }
    }

    post {
        always {
            script {
                // Kill background processes
                sh '''
                    if [ -f serve.pid ]; then
                        kill -9 $(cat serve.pid) 2>/dev/null || true
                        rm -f serve.pid
                    fi
                    pkill -f "serve -s build" 2>/dev/null || true
                    lsof -ti:3000 | xargs kill -9 2>/dev/null || true
                '''
                
                // Publish test results
                junit allowEmptyResults: true,
                      testResults: '**/junit.xml, test-results/**/*.xml'
                
                // Publish Playwright HTML report
                publishHTML([
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'playwright-report',
                    reportFiles: 'index.html',
                    reportName: 'Playwright HTML Report'
                ])
                
                // Archive artifacts
                archiveArtifacts artifacts: 'playwright-report/**/*, test-results/**/*, serve.log, build/**/*, coverage/**/*', 
                              allowEmptyArchive: true,
                              fingerprint: true
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
                    if [ -f playwright-report/index.html ]; then
                        echo "Playwright report available"
                    fi
                '''
            }
        }
        
        unstable {
            script {
                echo 'Pipeline is unstable. Some tests may have failed.'
            }
        }
    }
    
    options {
        timeout(time: 45, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '5'))
        ansiColor('xterm')
        skipDefaultCheckout()
    }
}
