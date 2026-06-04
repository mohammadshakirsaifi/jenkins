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
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
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
                                        npm test
                                    '''
                                }
                            }
                        },

                        "E2E Tests": {
                            node {
                                docker.image('mcr.microsoft.com/playwright:v1.39.0-jammy').inside {
                                    sh '''
                                        npm ci
                                        npm install serve

                                        serve -s build &
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
                    junit allowEmptyResults: true, testResults: 'jest-results/junit.xml'

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
