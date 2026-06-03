pipeline {
    agent any

    environment {
        NPM_CONFIG_CACHE = "/tmp/.npm"
    }

    stages {

        stage('Install Dependencies') {
            agent {
                docker {
                    image 'node:18-bullseye'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    npm config set cache /tmp/.npm --global
                    npm install
                '''
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'node:18-bullseye'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    npm config set cache /tmp/.npm --global
                    CI=true npm test
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
                    npm config set cache /tmp/.npm --global

                    # start app (make sure build exists)
                    npx serve -s build &
                    sleep 10

                    # run playwright tests
                    npx playwright test
                '''
            }
        }
    }

    post {
        always {
            script {
                if (fileExists('jest-results/junit.xml')) {
                    junit 'jest-results/junit.xml'
                } else {
                    echo "JUnit report not found"
                }
            }
        }
    }
}
