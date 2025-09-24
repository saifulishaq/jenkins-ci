pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        AWS_DEFAULT_REGION = "eu-west-2"
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

                        aws ecs update-service --cluster learn-jenkins-prod --service learn-jenkins-app-prod-service-1 --task-definition learn-jenkins-app-prod:$LATEST_TD_REVISION
                    '''
                }
            }
        }
    }
}
