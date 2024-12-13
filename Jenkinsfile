pipeline {
    agent {
        docker {
            image 'jenkins/jenkins:lts'
        }
    }
    environment {
        KUBECTL_IMAGE = 'k8s.gcr.io/kubectl:v1.24.0'
        HELM_IMAGE = 'helm/helm:v3.10.0'
    }
    stages {
        stage('Build Docker Image') {
            steps {
                docker.build 'saicherry93479/html-deploy:latest', '.'
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh '''
                    docker run --rm -v /path/to/kubeconfig:/root/.kube/config -v /var/run/docker.sock:/var/run/docker.sock -e DOCKER_BUILDKIT=1 ${KUBECTL_IMAGE} apply -f k8s/deployment.yaml
                    '''
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
