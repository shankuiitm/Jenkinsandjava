pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/shankuiitm/Jenkinsandjava.git'
        AWS_REGION = 'us-east-1'
        ECR_PUBLIC_REPO_URI = 'public.ecr.aws/l4g0s5q6/jenkinsecr'
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

       stage('Create ECR Repo') {
    steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
            sh '''
                aws ecr-public describe-repositories \
                --region us-east-1 \
                --repository-names jenkinsecr || \

                aws ecr-public create-repository \
                --repository-name jenkinsecr \
                --region us-east-1
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

