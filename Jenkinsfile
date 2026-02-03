pipeline {
    agent any

    environment {
        AWS_REGION             = 'us-east-1'
        AWS_ACCESS_KEY_ID      = credentials('Access Key ID')       // Your AWS access key stored in Jenkins
        AWS_SECRET_ACCESS_KEY  = credentials('Secret Access Key')  // Your AWS secret key stored in Jenkins
        ECR_REPO               = '529589763090.dkr.ecr.us-east-1.amazonaws.com/my-app-repo'
        ECS_CLUSTER            = 'arn:aws:ecs:us-east-1:529589763090:cluster/my-ecs-cluster'
        ECS_SERVICE            = 'my-app-service'
        IMAGE_TAG              = "${env.BUILD_NUMBER}" // unique tag per build
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t my-app:${IMAGE_TAG} .'
            }
        }

        stage('Authenticate to ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${ECR_REPO}
                """
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                sh """
                    docker tag my-app:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}
                    docker push ${ECR_REPO}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy to ECS') {
            steps {
                sh """
                    echo "Deploying image ${ECR_REPO}:${IMAGE_TAG} to ECS..."
                    aws ecs update-service \
                        --cluster ${ECS_CLUSTER} \
                        --service ${ECS_SERVICE} \
                        --force-new-deployment \
                        --region ${AWS_REGION} \
                        --image ${ECR_REPO}:${IMAGE_TAG}
                """
            }
        }
    }

    post {
        success {
            echo "Deployment completed successfully!"
        }
        failure {
            echo "Deployment failed!"
        }
    }
}
