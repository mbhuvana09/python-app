pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO_URL = '044274180134.dkr.ecr.us-east-1.amazonaws.com/python-app'
        IMAGE_TAG = 'latest'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh "docker build -t python-app ."
                }
            }
        }

        stage('Tag Docker Image') {
            steps {
                script {
                    sh "docker tag python-app:latest ${ECR_REPO_URL}:${IMAGE_TAG}"
                }
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([aws(credentialsId: 'your-aws-credentials-id', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    script {
                        sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO_URL}"
                    }
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    // Push the Docker image to ECR
                    sh "docker push ${ECR_REPO_URL}:${IMAGE_TAG}"
                }
            }
        }
    }

    post {
        success {
            echo 'Docker image pushed to ECR successfully!'
        }
        failure {
            echo 'Failed to push Docker image to ECR.'
        }
    }
}
