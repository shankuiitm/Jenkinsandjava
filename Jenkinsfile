pipeline {
    agent { label 'java-agent' }

    environment {
        GIT_REPO = 'https://github.com/abhimangowdagr/Jenkinsandjava.git'
        AWS_REGION = 'ap-south-1'
        ECR_PUBLIC_REPO_URI = 'public.ecr.aws/q0e7m1l2/jenkinsecr'
        IMAGE_TAG = 'latest'
        IMAGE_URI = "${ECR_PUBLIC_REPO_URI}:${IMAGE_TAG}"
        EKS_CLUSTER = 'my-eks-cluster'
    }

    stages {

        stage('Clone Repository') {
            steps {
                git url: "${GIT_REPO}", branch: 'main'
            }
        }

        stage('Build Application') {
            steps {
                sh '''
                echo "Building Java application..."
                mvn clean package
                '''
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

        stage('Login to AWS ECR') {
            steps {
                sh '''
                echo "Logging into AWS ECR..."
                aws ecr-public get-login-password --region us-east-1 \
                | docker login --username AWS --password-stdin public.ecr.aws
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                echo "Pushing Docker image..."
                docker push ${IMAGE_URI}
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                echo "Updating kubeconfig..."
                aws eks update-kubeconfig --region $AWS_REGION --name $EKS_CLUSTER

                echo "Deploying application..."
                kubectl apply -f deploymentjava.yaml
                kubectl apply -f servicelb.yaml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                kubectl get pods
                kubectl get svc
                '''
            }
        }
    }

    post {
        success {
            echo "🚀 Deployment Successful!"
        }
        failure {
            echo "❌ Pipeline Failed!"
        }
    }
}
