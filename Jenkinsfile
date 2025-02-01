pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = "harbor.local"
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
                    sh 'mvn clean package -DskipTests' // Compile l'application
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_REGISTRY}/microservice/${IMAGE_NAME}:${GIT_COMMIT}")
                }
            }
        }

        stage('Push to Harbor') {
            steps {
                script {
                    docker.withRegistry("http://${DOCKER_REGISTRY}", 'harbor-credentials') {
                        docker.image("${DOCKER_REGISTRY}/microservice/${IMAGE_NAME}:${GIT_COMMIT}").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    kubernetesDeploy(
                        kubeconfigId: 'kubeconfig',
                        configs: 'kubernetes/deployment.yaml',
                        enableConfigSubstitution: true,
                        secretName: 'my-secret'
                    )
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
