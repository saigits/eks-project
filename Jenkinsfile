pipeline {
    agent any
    environment {
        // GitHub Repo and Branch details
        GIT_REPO = 'https://github.com/saigits/eks-project.git'
        GIT_BRANCH = 'main'

        // ECR Details
        ECR_REGISTRY = '711387117319.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REPO = 'my-eks-repo'
        IMAGE_NAME = 'my-eks-image'

        // Kubernetes Details
        K8S_NAMESPACE = 'sai'
        K8S_SERVICE_NAME = 'sai-eks'

        // AWS CLI
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '711387117319' // Add your AWS Account ID here
    }
    stages {
        stage('Checkout Code') {
            steps {
                // Checkout the code from the GitHub repository
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO}"
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image using the Dockerfile in the repository
                    //docker.build("${IMAGE_NAME}:latest")
                    sh '''
                      docker build -t ${IMAGE_NAME}:latest .
                       '''
                }
            }
        }
        stage('Login to ECR') {
            steps {
                script {
                    // Log in to AWS ECR using the AWS CLI
                    sh '''
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    '''
                }
            }
        }
        stage('Tag and Push Docker Image to ECR') {
            steps {
                script {
                    // Tag the Docker image and push it to ECR
                    sh '''
                        docker tag ${IMAGE_NAME}:latest ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_NAME}-latest
                        docker push ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_NAME}-latest
                    '''
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Deploy the image to the EKS cluster
                    sh '''
                        kubectl config use-context arn:aws:eks:${AWS_REGION}:${AWS_ACCOUNT_ID}:cluster/my-eks-cluster
                        kubectl apply -f k8s/deployment.yaml --namespace=${K8S_NAMESPACE}
                        kubectl apply -f k8s/service.yaml --namespace=${K8S_NAMESPACE}
                    '''
                }
            }
        }
    }
    post {
        failure {
            // Clean up Docker resources if any stage fails
            sh 'docker system prune -f'
        }
    }
}
