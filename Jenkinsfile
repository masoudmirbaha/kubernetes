pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIAL = 'docker-hub-creds'   // نام Credential در Jenkins
        DOCKER_IMAGE = 'masoudmirbaha/myapp'        // نام Docker image
        SERVER_USER = 'ubuntu'                       // یوزر سرور Ubuntu
        SERVER_HOST = '192.168.9.71'                // آی‌پی سرور Ubuntu
        SSH_CREDENTIAL = 'ssh-server-creds'         // Credential SSH در Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checkout code from GitHub...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    sh "docker build -t ${DOCKER_IMAGE}:latest ."
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    echo "Logging in to Docker Hub..."
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIAL}", usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh "docker push ${DOCKER_IMAGE}:latest"
                    }
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                script {
                    echo "Deploying application to server..."
                    withCredentials([sshUserPrivateKey(credentialsId: "${SSH_CREDENTIAL}", keyFileVariable: 'SSH_KEY')]) {
                        sh """
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no ${SERVER_USER}@${SERVER_HOST} '
                            docker pull ${DOCKER_IMAGE}:latest &&
                            docker stop myapp || true &&
                            docker rm myapp || true &&
                            docker run -d --name myapp -p 8080:8080 ${DOCKER_IMAGE}:latest
                        '
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs!'
        }
    }
}
