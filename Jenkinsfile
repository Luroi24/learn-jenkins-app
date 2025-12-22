pipeline {
    agent{
        docker{
            image 'node:18-alpine'
            reuseNode true
        }
    }

    stages {
        stage('Build') {
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

        stage('Test') {
            // Go into build folder and check that there is an index.html
            steps{
                sh '''
                    test build/build.html
                    npm test
                '''
            }
        }
    }
}
