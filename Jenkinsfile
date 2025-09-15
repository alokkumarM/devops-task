pipeline {
    agent any

    environment {
        
        AWS_ACCOUNT_ID = "864981754239" 
        AWS_DEFAULT_REGION = "ap-south-1" 
        
        ECR_REPOSITORY_NAME = "devops-task-app"
        AWS_CREDS = credentials('aws-credentials')
    }

    stages {
        stage('Build and Test') {
            steps {
                echo 'Installing dependencies and running tests...'
                sh 'npm install'
                sh 'npm test'
            }
        }
        stage('Build Docker Image') {
            steps {
                echo 'Building the Docker image...'
                script {
                    def imageName = "${ECR_REPOSITORY_NAME}:${env.BUILD_NUMBER}"
                    env.IMAGE_NAME = imageName
                    sh "docker build -t ${imageName} ."
                }
            }
        }
        stage('Push to AWS ECR') {
            steps {
                echo 'Pushing Docker image...'
                script {
                    sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                    sh "docker tag ${env.IMAGE_NAME} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${env.IMAGE_NAME}"
                    sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${env.IMAGE_NAME}"
                }
            }
        }
        stage('Deploy on EC2') {
            steps {
                echo 'Deploying new container...'
                script {
                    def imageUrl = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${env.IMAGE_NAME}"
                    // Stop and remove the old container to avoid port conflicts
                    sh "docker stop devops-task-app-container || true"
                    sh "docker rm devops-task-app-container || true"
                    // Run the new container from the image in ECR
                    sh "docker run -d -p 3000:3000 --name devops-task-app-container ${imageUrl}"
                }
            }
        }
    }
}

