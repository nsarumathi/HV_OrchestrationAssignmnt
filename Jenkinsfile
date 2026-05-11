pipeline {
    agent any

    environment {

        AWS_REGION = 'ap-south-2'
        AWS_ACCOUNT_ID = '944765969321'

        FRONTEND_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/hv-frontend"

        ADMIN_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/hv-adminservice"

        AUTH_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/hv-authservice"

        CHAT_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/hv-chatservice"

        STREAM_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/hv-streamingservice"

        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Clone Source Code') {
            steps {

                git branch: 'main',
                url: 'https://github.com/nsarumathi/HV_OrchestrationAssignmnt.git'
            }
        }

        stage('Docker Compose Build') {
            steps {

                  sh '''
                docker compose build adminservice authservice chatservice streamingservice
                '''
            }
        }

        stage('AWS ECR Login') {
            steps {

                sh '''
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                '''
            }
        }

        stage('Tag Docker Images') {
            steps {

                sh '''

                docker tag frontend:latest $FRONTEND_REPO:$IMAGE_TAG

                docker tag hv_orchestrationpipeline-adminservice:latest $ADMIN_REPO:$IMAGE_TAG

                docker tag hv_orchestrationpipeline-authservice:latest $AUTH_REPO:$IMAGE_TAG

                docker tag hv_orchestrationpipeline-chatservice:latest $CHAT_REPO:$IMAGE_TAG

                docker tag hv_orchestrationpipeline-streamingservice:latest $STREAM_REPO:$IMAGE_TAG

                '''
            }
        }

        stage('Push Images To ECR') {
            steps {

                sh '''

                docker push $FRONTEND_REPO:$IMAGE_TAG

                docker push $ADMIN_REPO:$IMAGE_TAG

                docker push $AUTH_REPO:$IMAGE_TAG

                docker push $CHAT_REPO:$IMAGE_TAG

                docker push $STREAM_REPO:$IMAGE_TAG

                '''
            }
        }
    }

    post {

        success {

            echo 'All Docker Images Successfully Pushed To ECR'
        }

        failure {

            echo 'Pipeline Failed'
        }
    }
}
