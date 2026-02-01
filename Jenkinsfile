pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.2.$BUILD_ID"
        APP_NAME = "myjenkinsapp"
        AWS_REGION = "us-east-1"
        AWS_ECS_CLUSTER = "lern-jenkins-app-kd9t13"
        AWS_ECS_SERVICE = "LearnJemkinsApp-TaskDefination-Prod-service-m0nsrhfu"
        AWS_ECS_TASK_DEFINITION = "LearnJemkinsApp-TaskDefination-Prod"
        AWS_ECR_REPOSITORY = "595199363770.dkr.ecr.us-east-1.amazonaws.com"
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

        stage('Build Docker image') {
            agent {
                docker {
                    image 'my-aws-cli'  
                    args '-u root -v /var/run/docker.sock:/var/run/docker.sock  --entrypoint=""'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    docker build -t $AWS_ECR_REPOSITORY/$APP_NAME:$REACT_APP_VERSION .
                    aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_ECR_REPOSITORY
                    docker push $AWS_ECR_REPOSITORY/$APP_NAME:$REACT_APP_VERSION
                '''
            }
        }

        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'my-aws-cli'
                    args '--entrypoint=""'
                    reuseNode true
                }
            }
            environment {
                NEW_REVISION = ""
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version

                        NEW_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json --region $AWS_REGION | jq '.taskDefinition.revision')
                        echo $NEW_REVISION
                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE --task-definition $AWS_ECS_TASK_DEFINITION:$NEW_REVISION
                        aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE
                    '''
                }
            }
        }

        
    }
}
