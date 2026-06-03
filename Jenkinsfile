pipeline {
  agent any

  environment {
    NPM_CONFIG_CACHE = "${WORKSPACE}/.npm"
  }

  stages {

    stage('Install & Test') {
      agent {
        docker { image 'node:18-alpine' }
      }
      steps {
        sh '''
          mkdir -p .npm
          npm ci
          npm test -- --watchAll=false
        '''
      }
    }

    stage('Build') {
      agent {
        docker { image 'node:18-alpine' }
      }
      steps {
        sh '''
          npm ci
          npm run build
        '''
      }
    }

    stage('E2E Tests (Playwright)') {
      agent {
        docker {
          image 'mcr.microsoft.com/playwright:v1.60.0-jammy'
          args '--ipc=host'
        }
      }
      steps {
        sh '''
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
      junit 'test-results/**/*.xml'
      archiveArtifacts artifacts: 'build/**', fingerprint: true
    }
  }
}
