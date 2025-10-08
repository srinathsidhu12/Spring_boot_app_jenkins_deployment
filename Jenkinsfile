pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub-id') // Jenkins credentials ID
        DOCKER_HUB_REPO = "<dockerhub-username>/springboot-demo"
        K8S_DEPLOYMENT_NAME = "springboot-demo"
        K8S_CONTAINER_NAME = "springboot-demo"
    }

    triggers {
        githubPush() // Webhook trigger for code changes
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/<your-username>/springboot-demo.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    env.IMAGE_TAG = "${env.BUILD_NUMBER}"
                    docker.build("${DOCKER_HUB_REPO}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'DOCKER_HUB_CREDENTIALS') {
                        docker.image("${DOCKER_HUB_REPO}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                    kubectl set image deployment/${K8S_DEPLOYMENT_NAME} \
                    ${K8S_CONTAINER_NAME}=${DOCKER_HUB_REPO}:${IMAGE_TAG} --record
                    kubectl rollout status deployment/${K8S_DEPLOYMENT_NAME}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Successfully deployed image: ${DOCKER_HUB_REPO}:${IMAGE_TAG}"
        }
        failure {
            echo "❌ Deployment failed!"
        }
    }
}

