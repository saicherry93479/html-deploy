pipeline {
    agent any

    environment {
        EKS_CLUSTER_NAME = 'your-eks-cluster-name'
        AWS_REGION = 'us-east-2'
        DOCKER_IMAGE = 'saicherry93479/html-deploy'
        BUILD_NUMBER = "${env.BUILD_NUMBER ?: 'latest'}"
    }

    stages {
        stage('Docker Build & Push') {
            steps {
                script {
                    // Build Docker image for HTML deployment
                    def dockerImage = docker.build(
                        "${DOCKER_IMAGE}:${BUILD_NUMBER}",
                        "."
                    )

                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Configure Kubectl') {
            steps {
                withAWS(credentials: 'AWS-CREDS', region: "${AWS_REGION}") {
                    sh """
                        aws eks update-kubeconfig \
                            --name ${EKS_CLUSTER_NAME} \
                            --region ${AWS_REGION}
                    """
                }
            }
        }

        stage('Prepare Deployment Files') {
            steps {
                script {
                    // Replace the build number in the deployment file
                    sh """
                        sed -i 's|\${BUILD_NUMBER}|${BUILD_NUMBER}|g' k8s/deployment.yaml
                    """
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withAWS(credentials: 'AWS-CREDS', region: "${AWS_REGION}") {
                    // Apply Kubernetes manifests for HTML deployment
                    sh """
                        kubectl apply -f k8s/html-deployment.yaml
                        kubectl apply -f k8s/service.yaml

                        # Wait for the deployment to roll out successfully
                        kubectl rollout status deployment/html-deploy
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Build, push, and deployment to EKS successful for HTML deploy!'
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            // Clean up the Kubernetes context
            sh 'kubectl config unset current-context'
        }
    }
}
