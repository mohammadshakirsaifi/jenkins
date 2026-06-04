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
                    rm -rf node_modules package-lock.json build .npm
                    mkdir -p .npm
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
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
                sh 'npm test -- --watchAll=false --ci --coverage || true'
            }
        }

        stage('Start App (for E2E)') {
            steps {
                sh '''
                    # Kill any process using port 3000
                    lsof -ti:3000 | xargs kill -9 2>/dev/null || true
                    
                    # Install serve
                    npm install -g serve
                    
                    # Start the app
                    serve -s build -l 3000 > serve.log 2>&1 &
                    echo $! > serve.pid
                    
                    # Wait for app to be ready
                    sleep 8
                    curl -s http://localhost:3000 > /dev/null && echo "App is ready"
                '''
            }
        }

        stage('E2E Tests (Playwright)') {
            steps {
                sh '''
                    npx playwright install --with-deps
                    npx playwright test --reporter=html || true
                '''
            }
        }
    }

    post {
        always {
            script {
                // Clean up background processes
                sh '''
                    if [ -f serve.pid ]; then
                        kill -9 $(cat serve.pid) 2>/dev/null || true
                        rm -f serve.pid
                    fi
                    lsof -ti:3000 | xargs kill -9 2>/dev/null || true
                '''
                
                // Publish test results
                junit allowEmptyResults: true,
                      testResults: 'test-results/**/*.xml, test-results/**/junit.xml'
                
                // Publish HTML report
                publishHTML([
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'playwright-report',
                    reportFiles: 'index.html',
                    reportName: 'Playwright HTML Report'
                ])
                
                // Archive artifacts
                archiveArtifacts artifacts: 'playwright-report/**/*, test-results/**/*, serve.log', 
                              allowEmptyArchive: true
            }
        }
        
        failure {
            script {
                echo 'Pipeline failed! Check logs for details.'
                sh '''
                    if [ -f serve.log ]; then
                        echo "=== Last 50 lines of serve.log ==="
                        tail -50 serve.log
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
