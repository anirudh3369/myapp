pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "anirud3369/myapp"
        // Use forward slashes to avoid Groovy escaping issues
        KUBE_CONFIG = "C:/ProgramData/.kube/config"
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Explicitly specify branch and credentials to avoid errors
                git branch: 'main', url: 'https://github.com/anirudh3369/myapp.git', credentialsId: 'github-creds'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:latest")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-creds') {
                        docker.image("${DOCKER_IMAGE}:latest").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // Use 'bat' on Windows instead of 'sh'
                bat "kubectl --kubeconfig=${KUBE_CONFIG} apply -f k8s/deployment.yaml"
                bat "kubectl --kubeconfig=${KUBE_CONFIG} apply -f k8s/service.yaml"
            }
        }
    }

    post {
        success {
            echo "Deployment completed successfully!"
        }
        failure {
            echo "Deployment failed!"
        }
    }
}
