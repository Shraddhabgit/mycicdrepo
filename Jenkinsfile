pipeline {
    agent any

    environment {
        // Replace with your AWS ECR details
        AWS_REGION = "ap-south-1"
        ECR_REPO = "383856007426.dkr.ecr.ap-south-1.amazonaws.com/mywar-app"
        IMAGE_TAG = "latest"
        K8S_DEPLOYMENT = "mywar-deployment"
        K8S_NAMESPACE = "default"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Shraddhabgit/mycicdrepo.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
            }
        }

        stage('Docker Push to ECR') {
            steps {
                script {
                    // Authenticate Docker to ECR
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}"

                    // Push image
                    sh "docker push ${ECR_REPO}:${IMAGE_TAG}"
                }
            }
        }
        stage('Deploy to EKS') {
            steps {
                script {
                    // Update kubeconfig for EKS cluster
                    sh "aws eks update-kubeconfig --region ${AWS_REGION} --name education-eks-rh75QaKN"

                    // Update Kubernetes deployment with new image
                    sh """
                        kubectl set image deployment/${K8S_DEPLOYMENT} \
                        mywar-container=${ECR_REPO}:${IMAGE_TAG} \
                        -n ${K8S_NAMESPACE}
                    """

                    // Verify rollout
                    sh "kubectl rollout status deployment/${K8S_DEPLOYMENT} -n ${K8S_NAMESPACE}"
                }
            }
        }
        stage('Apply Service') {
            steps {
                script {
                    // Apply the service manifest to ensure service is present
                    sh "kubectl apply -f service.yaml -n ${K8S_NAMESPACE}"
                }
            }
        }

    }
}
