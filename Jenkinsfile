pipeline {
agent any

```
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

                # Use workspace-local cache
                export HOME=$WORKSPACE
                export NPM_CONFIG_CACHE=$WORKSPACE/.npm

                # Clean any previous artifacts
                rm -rf node_modules

                npm ci
                npm run build

                ls -la
            '''
        }
    }
}
```

}
