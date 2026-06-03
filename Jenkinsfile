pipeline {
    agent any

    stages {
        /*

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
					export HOME=$WORKSPACE
                    export NPM_CONFIG_CACHE=$WORKSPACE/.npm
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        */

        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    #test -f build/index.html
                    npm test
                '''
            }
        }

        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.60.0-noble'
                    reuseNode true
                }
            }

            steps {
                sh '''
					export HOME=$WORKSPACE
                    export NPM_CONFIG_CACHE=$WORKSPACE/.npm
					npm ci
                    npx serve -s build -l 3000 &
                    npx wait-on http://localhost:3000
                    npx playwright test
                '''
            }
        }
    }

    post {
        always {
            junit 'jest-results/junit.xml'
        }
    }
}
