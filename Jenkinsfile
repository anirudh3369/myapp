pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "anirudh3369/myapp"
        // Use forward slashes to avoid Groovy escaping issues
        KUBE_CONFIG = "C:/ProgramData/.kube/config"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "Step 1: Cloning the repository..."
                git branch: 'main', url: 'https://github.com/anirudh3369/myapp.git', credentialsId: 'github-creds'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Step 2: Building the Docker image..."
                script {
                    docker.build("${DOCKER_IMAGE}:latest")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "Step 3: Pushing Docker image to DockerHub..."
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-creds') {
                        docker.image("${DOCKER_IMAGE}:latest").push()
                    }
                }
            }
        }

        stage('Determine Active Environment') {
            steps {
                echo "Step 4: Detecting which version (Blue/Green) is currently live..."
                script {
                    def currentVersion = bat(
                        script: "kubectl --kubeconfig=${KUBE_CONFIG} get svc myapp-service -o=jsonpath='{.spec.selector.version}'",
                        returnStdout: true
                    ).trim()

                    if (currentVersion == "blue") {
                        env.TARGET_COLOR = "green"
                        env.CURRENT_COLOR = "blue"
                    } else {
                        env.TARGET_COLOR = "blue"
                        env.CURRENT_COLOR = "green"
                    }

                    echo "🟦 Current live version: ${env.CURRENT_COLOR.toUpperCase()}"
                    echo "🟩 Deploying new version: ${env.TARGET_COLOR.toUpperCase()}"
                }
            }
        }

        stage('Deploy New Version') {
            steps {
                script {
                    echo "⚙️ Step 5: Deploying ${TARGET_COLOR.toUpperCase()} version to Kubernetes..."
                    bat "kubectl --kubeconfig=${KUBE_CONFIG} apply -f k8s/${TARGET_COLOR}-deployment.yaml"

                    echo "⏳ Waiting for ${TARGET_COLOR.toUpperCase()} pods to become ready..."
                    bat "kubectl --kubeconfig=${KUBE_CONFIG} rollout status deployment/pythonapp-${TARGET_COLOR}"
                }
            }
        }

        stage('Switch Traffic') {
            steps {
                script {
                    echo "🔄 Step 6: Switching service traffic from ${CURRENT_COLOR.toUpperCase()} ➜ ${TARGET_COLOR.toUpperCase()} ..."
                    bat "kubectl --kubeconfig=${KUBE_CONFIG} patch service myapp-service -p \"{\\\"spec\\\":{\\\"selector\\\":{\\\"version\\\":\\\"${TARGET_COLOR}\\\"}}}\""
                    echo "✅ Traffic now routed to ${TARGET_COLOR.toUpperCase()} deployment!"
                }
            }
        }

        stage('Clean Up Old Version') {
            steps {
                script {
                    echo "🧹 Step 7: Cleaning up old ${CURRENT_COLOR.toUpperCase()} deployment..."
                    def status = bat(
                        script: "kubectl --kubeconfig=${KUBE_CONFIG} delete deployment pythonapp-${CURRENT_COLOR}",
                        returnStatus: true
                    )
                    if (status != 0) {
                        echo "⚠️ Old ${CURRENT_COLOR.toUpperCase()} deployment not found — skipping cleanup."
                    } else {
                        echo "🗑️ Old ${CURRENT_COLOR.toUpperCase()} deployment deleted successfully."
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Blue-Green Deployment completed successfully!"
            echo "🌈 Live Version: ${env.TARGET_COLOR.toUpperCase()}"
        }
        failure {
            echo "❌ Deployment failed during ${env.TARGET_COLOR.toUpperCase()} rollout!"
        }
    }
}
