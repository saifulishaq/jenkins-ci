pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        AWS_DEFAULT_REGION = "eu-west-2"
        AWS_ECS_CLUSTER = "learn-jenkins-prod"
        AWS_ECS_SERVICE_PROD = "learn-jenkins-app-prod-service-1"
        AWS_ECS_TD = "learn-jenkins-app-prod"
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
                    npm ci 
                    npm run build
                '''
            }
        }
        stage('Build application image'){
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                    reuseNode true
                }
            }
            steps{
                sh '''
                    amazon-linux-extras install docker
                    docker build -t my-app .

                '''
            }
        }
        stage('AWS ECS Deployment'){
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "-u root --entrypoint=''"
                    reuseNode true
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        yum install jq -y

                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://./aws/task-definition-prod.json | jq '.taskDefinition.revision')

                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE_PROD --task-definition $AWS_ECS_TD:$LATEST_TD_REVISION

                    '''
                }
            }
        }
    }
}
