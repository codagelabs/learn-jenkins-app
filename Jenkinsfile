pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '3dc6f51c-7d26-40e3-8b54-5275e0a93557'
        NETLIFY_AUTH_TOKEN = credentials('netify-token') 
        REACT_APP_VERSION = "1.2.$BUILD_ID"
    }
    
    stages {

        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args '--entrypoint=""'
                }
            }
            environment {
                AWS_BUCKET = 'learn-jenkins-app-2026'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        echo "Hello World" > index.html
                        aws s3 cp index.html s3://$AWS_BUCKET/index.html --region ap-south-1 
                    '''
                }
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Building the app ..."
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
                            #test -f build/index.html
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
                            image 'my-playwright-app'  
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            serve -s build &
                            sleep 10
                            npx playwright test  --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        
        stage('STAGE Deploy') {
            agent {
                docker {
                    image 'my-playwright-app'  
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = ' Not set'
            }

            steps {
                sh '''
                    node --version
                    netlify --version
                    echo "Deploying to Netlify site id $NETLIFY_SITE_ID ..."
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                    npx playwright test  --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Deploy Prod') {
            agent {
                docker {
                    image 'my-playwright-app'  
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL='https://dynamic-parfait-b5c1dd.netlify.app'
            }

            steps {
                sh '''
                    npm --version
                    node --version
                    netlify --version
                    echo "Deploying to Netlify site id $NETLIFY_SITE_ID ..."
                    netlify status
                    netlify deploy --dir=build --prod
                    npx playwright test  --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}
