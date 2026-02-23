pipeline {
    agent { label 'Java_Agent1' }

    environment {
        AWS_REGION = 'us-east-1'
        ECR_PUBLIC_REPO_URI = 'public.ecr.aws/l4g0s5q6/test-project'
        IMAGE_TAG = 'latest'
        IMAGE_URI = "${ECR_PUBLIC_REPO_URI}:${IMAGE_TAG}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Java App') {
            steps {
                sh '''
                    echo "Building Java application..."
                    mvn clean -B -Denforcer.skip=true package
                '''
            }
        }

        stage('Login to AWS ECR Public') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds'
                ]]) {
                    sh '''
                        echo "Configuring AWS CLI..."
                        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                        aws configure set region ${AWS_REGION}

                        echo "Logging in to ECR Public..."
                        aws ecr-public get-login-password --region us-east-1 \
                        | docker login --username AWS --password-stdin public.ecr.aws
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building Docker image..."
                    docker build -t ${IMAGE_URI} .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    echo "Pushing Docker image to ECR Public..."
                    docker push ${IMAGE_URI}
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Image pushed successfully: ${IMAGE_URI}"
        }
        failure {
            echo "❌ Pipeline failed. Check logs above."
        }
    }
}
