pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'mynameaom/test_deploy'  // ชื่อ image บน Docker Hub
        DOCKER_TAG = 'latest'
        DOCKER_HUB_USER = 'mynameaom'  // Username ของ Docker Hub
        DOCKER_HUB_CREDENTIALS = 'c8ad2cd2-59b2-4f96-8980-162d28142755' // Credentials ID ใน Jenkins
        SERVER_USER = 'root'
        SERVER_HOST = '192.168.136.134'
        SSH_CREDENTIALS_ID = '6e628bfe-ef3d-4e27-9237-2465ebf4bb97' // เปลี่ยนเป็น Credentials ID ของคุณ
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "Checking out the code"
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker Image"
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    echo "Pushing Docker Image to Docker Hub"
                    withCredentials([usernamePassword(credentialsId: DOCKER_HUB_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh """
                            echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USER} --password-stdin
                            docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                        """
                    }
                }
            }
        }

       stage('Deploy to Server') {
            steps {
                script {
                    echo "Deploying Application on Server"
                    withCredentials([sshUserPrivateKey(credentialsId: SSH_CREDENTIALS_ID, keyFileVariable: 'SSH_KEY')]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} ${SSH_USER}@${SERVER_HOST} << 'EOF'
                            echo 'Stopping and removing existing container...'
                            docker stop react-app-container || true
                            docker rm react-app-container || true

                            echo 'Pulling latest image...'
                            docker pull mynameaom/test_deploy:latest

                            echo 'Running new container...'
                            docker run -d --name react-app-container -p 80:80 mynameaom/test_deploy:latest

                            echo 'Removing unused images...'
                            docker rmi mynameaom/test_deploy:latest || true
                            EOF
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Deployment completed successfully!"
        }
        failure {
            echo "Deployment failed. Check the logs for more details."
        }
    }
}
