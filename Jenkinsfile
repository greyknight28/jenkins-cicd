pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        ECR_REPO = "300601068948.dkr.ecr.ap-south-1.amazonaws.com/jenkins_repo"
        IMAGE_TAG = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/greyknight28/jenkins-cicd.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${ECR_REPO}:${IMAGE_TAG}")
                }
            }
        }

        stage('Login to ECR') {
            steps {
                script {
                    sh '''
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin $ECR_REPO
                    '''
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    sh "docker push ${ECR_REPO}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    sh '''
                    aws eks update-kubeconfig --name my_eks_cluster --region $AWS_REGION
                    kubectl apply -f Deployment.yaml
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Deployment successful! Check your LoadBalancer URL."
        }
        failure {
            echo "❌ Deployment failed."
        }
    }
}
