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

        stage('Inject Env for Build') {
            steps {
                sh '''
                export MONGO_URI=$MONGO_URI
                export JWT_SECRET=$JWT_SECRET
                export CLIENT_URLS=$CLIENT_URLS
                export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                export AWS_REGION=$AWS_REGION
                export AWS_S3_BUCKET=$AWS_S3_BUCKET
                export AWS_CDN_URL=$AWS_CDN_URL
                export STREAMING_PUBLIC_URL=$STREAMING_PUBLIC_URL
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                docker compose build
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

        stage('Tag Images') {
            steps {
                sh '''
                docker tag hv-authservice:latest $AUTH_REPO:$IMAGE_TAG
                docker tag hv-adminservice:latest $ADMIN_REPO:$IMAGE_TAG
                docker tag hv-chatservice:latest $CHAT_REPO:$IMAGE_TAG
                docker tag hv-streamingservice:latest $STREAM_REPO:$IMAGE_TAG
                docker tag hv-frontend:latest $FRONTEND_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Push Images to ECR') {
            steps {
                sh '''
                docker push $AUTH_REPO:$IMAGE_TAG
                docker push $ADMIN_REPO:$IMAGE_TAG
                docker push $CHAT_REPO:$IMAGE_TAG
                docker push $STREAM_REPO:$IMAGE_TAG
                docker push $FRONTEND_REPO:$IMAGE_TAG
                '''
            }
        }
    }

    post {
        success {
            echo '✅ All Docker Images Successfully Pushed To ECR'
        }
        failure {
            echo '❌ Pipeline Failed'
        }
    }
}
