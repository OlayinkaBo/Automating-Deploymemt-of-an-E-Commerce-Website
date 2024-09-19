pipeline {
    agent any
    environment {
        DOCKER_REGISTRY = 'olayinkabo2' // DockerHub username
        IMAGE_NAME = 'nginx-app'
        TARGET_SERVER = '34.227.112.177'
        SSH_CREDENTIALS_ID = 'application-server-ssh'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    credentialsId: 'git-credentials', 
                    url: 'https://github.com/OlayinkaBo/Automating-Deploymemt-of-an-E-Commerce-Website.git'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    // Build the Docker image with the correct format
                    docker.build("${DOCKER_REGISTRY}/${IMAGE_NAME}:${env.BUILD_ID}")
                }
            }
        }

        stage('Push to Registry') {
            steps {
                script {
                    docker.withRegistry('', 'docker-registry-credentials') {
                        docker.image("${DOCKER_REGISTRY}/${IMAGE_NAME}:${env.BUILD_ID}").push()
                    }
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                script {
                    sshagent([SSH_CREDENTIALS_ID]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no user@${TARGET_SERVER} \\
                        'docker pull ${DOCKER_REGISTRY}/${IMAGE_NAME}:${env.BUILD_ID} && \\
                        docker stop ${IMAGE_NAME} || true && \\
                        docker rm ${IMAGE_NAME} || true && \\
                        docker run -d --name ${IMAGE_NAME} -p 80:80 ${DOCKER_REGISTRY}/${IMAGE_NAME}:${env.BUILD_ID}'
                        """
                    }
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
