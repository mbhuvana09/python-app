pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO_URL = '044274180134.dkr.ecr.us-east-1.amazonaws.com/python-app'
        IMAGE_TAG = 'latest'
        ECS_CLUSTER_NAME = 'python-app'
        ECS_SERVICE_NAME = 'python-app'
        TASK_FAMILY = 'family'
        CPU_TASK = '256'  // Task level CPU
        MEMORY_TASK = '512' // Task level Memory
        CPU_CONTAINER = '256'  // Container level CPU
        MEMORY_CONTAINER = '512' // Container level Memory
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
                    // Tag the Docker image
                    sh "docker tag python-app:latest ${ECR_REPO_URL}:${IMAGE_TAG}"
                }
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([aws(credentialsId: 'aws-creds', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    script {
                        // Log in to Amazon ECR
                        sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO_URL}"
                    }
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    // Push the Docker image to Amazon ECR
                    sh "docker push ${ECR_REPO_URL}:${IMAGE_TAG}"
                }
            }
        }

        stage('Register Task Definition') {
            steps {
                withCredentials([aws(credentialsId: 'aws-creds', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    script {
                        // Register a new task definition for ECS with the updated Docker image
                        sh """
                        aws ecs register-task-definition --family ${TASK_FAMILY} --network-mode awsvpc --cpu ${CPU_TASK} --memory ${MEMORY_TASK} --container-definitions '[{
                            "name": "python-app",
                            "image": "${ECR_REPO_URL}:${IMAGE_TAG}",
                            "memory": ${MEMORY_CONTAINER},
                            "cpu": ${CPU_CONTAINER},
                            "essential": true,
                            "portMappings": [{
                                "containerPort": 80,
                                "hostPort": 80
                            }]
                        }]' --requires-compatibilities FARGATE --execution-role-arn arn:aws:iam::044274180134:role/ecsTaskExecutionRole --region ${AWS_REGION}
                        """
                    }
                }
            }
        }

        stage('Update ECS Service') {
            steps {
                withCredentials([aws(credentialsId: 'aws-creds', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    script {
                        // Update the ECS service to use the latest task definition
                        sh "aws ecs update-service --cluster ${ECS_CLUSTER_NAME} --service ${ECS_SERVICE_NAME} --force-new-deployment --region ${AWS_REGION}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Docker image pushed to ECR, ECS Task Definition updated, and ECS Service redeployed successfully!'
        }
        failure {
            echo 'Failed to complete the deployment process.'
        }
    }
}
