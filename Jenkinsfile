pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '555044956356.dkr.ecr.us-east-1.amazonaws.com/ruhi-2048-app'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
    }

    stages {
        stage('Source') {
            steps {
                git branch: 'master', url: 'https://github.com/AlRuh-ux/ruhi-e2e-app-aws.git'
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t $ECR_REPO:$IMAGE_TAG .'
                sh 'aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO'
                sh 'docker push $ECR_REPO:$IMAGE_TAG'
            }
        }

        stage('Test') {
            steps {
                sh 'docker rm -f test-container || true'
                sh 'docker run -d --name test-container $ECR_REPO:$IMAGE_TAG'
                sh 'sleep 5'
                sh 'docker exec test-container wget -q -O- http://localhost:80 | grep -q "2048" && echo "Test passed"'
                sh 'docker stop test-container && docker rm test-container'
            }
        }

        stage('Deploy') {
            steps {
                sh 'helm upgrade ruhi-2048 ./helm/ruhi-2048 --namespace ruhi-2048 --set image.tag=$IMAGE_TAG'
            }
        }
    }
}