pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/shankuiitm/Jenkinsandjava.git'
        AWS_REGION = 'ap-south-1'
        ECR_PUBLIC_REPO_URI = 'public.ecr.aws/k0c8q8z5/jenkinsecr'
        IMAGE_TAG = 'latest'
        IMAGE_URI = "${ECR_PUBLIC_REPO_URI}:${IMAGE_TAG}"
        EKS_CLUSTER = 'my-eks-cluster'
    }

    stages {

        stage('Clone') {
            steps {
                git url: "${GIT_REPO}", branch: 'main'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('ECR Login') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh '''
                        aws ecr-public get-login-password --region us-east-1 \
                        | docker login --username AWS --password-stdin public.ecr.aws
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${IMAGE_URI} .'
            }
        }

        stage('Push Image') {
            steps {
                sh 'docker push ${IMAGE_URI}'
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh '''
                        aws eks update-kubeconfig --region $AWS_REGION --name $EKS_CLUSTER
                        kubectl apply -f deploymentjava.yaml
                        kubectl apply -f servicelb.yaml
                    '''
                }
            }
        }
    }
}

