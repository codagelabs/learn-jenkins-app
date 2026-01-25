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
                    node -v
                    npm -v
                    npm ci
                    npm run build
                    ls -la 

                '''
            }
            
        }
        stage("E2E TEST"){
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.58.0-noble'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Running E2E tests..."
                    npm install -g serve
                    serve -s build
                    npx playwright test
                '''
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Running tests..."
                    test build/index.html
                    npm test
                '''
            }
        }
    }
    post {
        always {
            junit 'test-results/*.xml'
        }
    }
}
