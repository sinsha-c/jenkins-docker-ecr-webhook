pipeline {
    agent any

    environment {
        AWS_REGION     = 'ap-south-1'
        ECR_LOGIN      = '489109585956.dkr.ecr.ap-south-1.amazonaws.com'
        ECR_REPO       = '489109585956.dkr.ecr.ap-south-1.amazonaws.com/nginx-webapp'
        IMAGE_TAG      = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t nginx-webapp:${IMAGE_TAG} ."
            }
        }

        stage('Smoke Test') {
            steps {
                sh """
                    docker run -d --name smoke-test -p 8081:80 nginx-webapp:${IMAGE_TAG}
                    sleep 5
                    curl -f http://localhost:8081 || exit 1
                    docker stop smoke-test && docker rm smoke-test
                """
            }
        }

        stage('Push to Amazon ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_LOGIN}
                    docker tag nginx-webapp:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}
                    docker push ${ECR_REPO}:${IMAGE_TAG}
                """
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully — image pushed to ECR.'
        }
        failure {
            echo 'Pipeline failed. Check the smoke test or ECR credentials.'
        }
    }
}
