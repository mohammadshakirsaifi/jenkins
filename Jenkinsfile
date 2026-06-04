pipeline {
    agent any

    stages {

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            environment {
                NPM_CONFIG_CACHE = "${WORKSPACE}/.npm"
            }

            steps {
                sh '''
                    mkdir -p $NPM_CONFIG_CACHE

                    npm ci --cache $NPM_CONFIG_CACHE --prefer-offline --no-audit --no-fund
                    npm run build
                '''
            }
        }

        stage('Tests') {
            steps {
                script {
                    parallel(

                        "Unit Tests": {
                            node {
                                docker.image('node:18-alpine').inside {
                                    sh '''
                                        mkdir -p .npm
                                        npm ci --cache .npm --prefer-offline --no-audit --no-fund
                                        npm test
                                    '''
                                }
                            }
                        },

                        "E2E Tests": {
                            node {
                                docker.image('mcr.microsoft.com/playwright:v1.39.0-jammy').inside {
                                    sh '''
                                        mkdir -p .npm
                                        npm ci --cache .npm --prefer-offline --no-audit --no-fund
                                        npm install serve

                                        npx serve -s build &
                                        sleep 10

                                        npx playwright test --reporter=html
                                    '''
                                }
                            }
                        }

                    )
                }
            }

            post {
                always {
                    junit allowEmptyResults: true,
                          testResults: 'jest-results/junit.xml'

                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: false,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Playwright HTML Report',
                        reportTitles: '',
                        useWrapperFileDirectly: true
                    ])
                }
            }
        }
    }
}
