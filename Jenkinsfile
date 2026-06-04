pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    export HOME=$WORKSPACE
                    npm config set cache $WORKSPACE/.npm-cache

                    rm -rf node_modules
                    npm ci --no-audit --no-fund
                    npm run build
                '''
            }
        }

        stage('Tests') {

            parallel {

                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            export HOME=$WORKSPACE
                            npm config set cache $WORKSPACE/.npm-cache

                            rm -rf node_modules
                            npm ci --no-audit --no-fund

                            npm test -- --watchAll=false
                        '''
                    }

                    post {
                        always {
                            junit '**/jest-results/*.xml'
                        }
                    }
                }

                stage('E2E Tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            export HOME=$WORKSPACE
                            npm config set cache $WORKSPACE/.npm-cache

                            rm -rf node_modules
                            npm ci --no-audit --no-fund

                            npx playwright install --with-deps
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Playwright HTML Report'
                            ])
                        }
                    }
                }
            }
        }
    }
}
