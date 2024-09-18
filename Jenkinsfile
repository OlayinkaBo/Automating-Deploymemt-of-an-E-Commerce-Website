pipeline {
    agent any
    environment {
        // Docker registry URL and image details
        DOCKER_REGISTRY = 'https://hub.docker.com/repositories/olayinkabo2'
        IMAGE_NAME = 'nginx-app'
        
        // Target server details
        TARGET_SERVER = '34.227.112.177'
        SSH_CREDENTIALS_ID = 'application-server-ssh'
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Checkout code from GitHub, specifying the correct branch (main in this case)
                git branch: 'main', // Replace 'main' with your actual branch name if different
                    credentialsId: 'git-credentials', 
                    url: 'https://github.com/OlayinkaBo/Automating-Deploymemt-of-an-E-Commerce-Website.git'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    // Build the Docker image and tag it with BUILD_ID
                    docker.build("${DOCKER_REGISTRY}/${IMAGE_NAME}:${env.BUILD_ID}")
                }
            }
        }

        stage('Push to Registry') {
            steps {
                script {
                    // Push Docker image to the registry with credentials
                    docker.withRegistry('', 'docker-registry-credentials') {
                        docker.image("${DOCKER_REGISTRY}/${IMAGE_NAME}:${env.BUILD_ID}").push()
                    }
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                script {
                    // SSH into the target server, stop, remove, and run the new Docker container
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
            // Clean up the workspace after the build
            cleanWs()
        }
    }
}
