pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '75f4fc9f-db50-4f46-ba28-4f0ff60577b0'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

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
                    echo 'small changess'
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
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo "Test stage"
                            test -f build/index.html
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
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 13
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright E2E Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }                             
                }                

            }
        }
        
        stage('Deploy + E2E Staging') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'VAR_TO_BE_SET'
            }

            steps {
                sh '''
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to staging. Site ID: ${NETLIFY_SITE_ID}"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json)
                    CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json                
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright E2E staging', reportTitles: '', useWrapperFileDirectly: true])
                }
            }                             
        }

        // stage('Approval') {
        //     steps {
        //         timeout(time: 15, unit: 'HOURS') {
        //             input message: 'Deploy to Production?', ok: 'Yes'
        //         }                          
        //     }
        // }

        stage('Deploy + E2E Prod') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://peppy-bunny-d8486e.netlify.app'
            }

            steps {
                sh '''
                    node --version
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to productin. Site ID: ${NETLIFY_SITE_ID}"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright E2E Prod', reportTitles: '', useWrapperFileDirectly: true])
                }
            }                             
        }  

    }
}
