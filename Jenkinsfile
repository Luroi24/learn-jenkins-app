pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'a9c7e4e3-c6f4-4d13-a3c3-dd6502d92bbd'
        NETLIFY_AUTH_TOKEN = credentials('token-netlify')
    }

    stages {
        // This is a comment
        stage('Build') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "simple message"
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Run Tests'){
            parallel{
                stage('Unit Test') {
                    agent{
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            test build/build.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.57.0-noble'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build  &
                            sleep 10
                            npx playwright test
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy Staging') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    
                '''
                script{
                    env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' netlify_build.json", returnStdout: true)
                }
            }
        }

        stage('Staging E2E') {
            environment{
                CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            }
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.57.0-noble'
                    reuseNode true
                }
            }
            steps{
                sh '''
                    npm install netlify-cli@20.1.1 node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > netlify_build.json
                    CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' netlify_build.json)
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Approval'){
            steps{
                timeout(time: 15, unit: 'MINUTES'){
                    input message:"Do you wish to deploy to production?", ok:"Yes, I am sure!"
                }
            }
        }

        stage('Deploy prod & Test') {
            environment{
                NETLIFY_SITE_ID = 'a9c7e4e3-c6f4-4d13-a3c3-dd6502d92bbd'
                CI_ENVIRONMENT_URL = 'https://meek-gecko-c43402.netlify.app'
                NETLIFY_AUTH_TOKEN = credentials('token-netlify')
            }
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.57.0-noble'
                    reuseNode true
                }
            }
            steps{
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod

                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}
