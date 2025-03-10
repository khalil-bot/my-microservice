pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = "docker.io/khaliloulah"
        IMAGE_NAME = "my-microservice"
        GIT_REPO = "https://github.com/khalil-bot/my-microservice.git"
        KUBE_CONTEXT = "kubernetes-admin@kubernetes"
    }
    tools {
        maven 'maven3'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: "${GIT_REPO}"
            }
        }

        stage('Build JAR') {
            steps {
                script {
                    sh 'mvn clean package -DskipTests' // Compile the application
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_REGISTRY}/${IMAGE_NAME}:v1")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-hub-credentials') {
                        docker.image("${DOCKER_REGISTRY}/${IMAGE_NAME}:v1").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh 'kubectl apply -f kubernetes/deployment.yaml'
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful'
        }
        failure {
            echo 'Deployment failed'
        }
    }
}
