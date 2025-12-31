pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'a9c7e4e3-c6f4-4d13-a3c3-dd6502d92bbd'
        NETLIFY_AUTH_TOKEN = credentials('token-netlify')
        REACT_APP_VERSION = "1.2.$BUILD_ID"
    }

    stages {
        stage('AWS'){
            agent{
                docker{
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'secret-aws-cli', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        echo "Hello S3!" > index.html
                        aws s3 cp index.html s3://luroi-jenkins-bucket 
                    '''
                }
            }
        }
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
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            serve -s build  &
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

        // stage('Deploy staging') {
        //     agent{
        
        //         docker{
        //             image 'my-playwright'
        //             reuseNode true
        //         }
        //     }

        //     environment{
        //         CI_ENVIRONMENT_URL = 'TBD'
        //     }

        //     steps{
        //         sh '''
        //             echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
        //             netlify status
        //             netlify deploy --dir=build --json > netlify_build.json
        //             CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' netlify_build.json)
        //             npx playwright test --reporter=html
        //         '''
        //     }
        //     post {
        //         always {
        //             publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        //         }
        //     }
        // }

        // stage('Deploy prod & Test') {
        //     environment{
        //         CI_ENVIRONMENT_URL = 'https://meek-gecko-c43402.netlify.app'
        //     }
        //     agent{
        //         docker{
        //             image 'my-playwright'
        //             reuseNode true
        //         }
        //     }
        //     steps{
        //         sh '''
        //             netlify --version
        //             echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
        //             netlify status
        //             netlify deploy --dir=build --prod

        //             npx playwright test --reporter=html
        //         '''
        //     }
        //     post {
        //         always {
        //             publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        //         }
        //     }
        // }
    }
}
