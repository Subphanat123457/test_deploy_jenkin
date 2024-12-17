pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'mynameaom/test_deploy'  // Docker image name
        DOCKER_TAG = 'latest'
        DOCKER_HUB_USER = 'mynameaom'  // Docker Hub username
        DOCKER_HUB_CREDENTIALS = 'c8ad2cd2-59b2-4f96-8980-162d28142755' // Docker Hub Credentials ID
        SERVER_USER = 'root'
        SERVER_HOST = '192.168.136.134'
        SSH_CREDENTIALS_ID = '6e628bfe-ef3d-4e27-9237-2465ebf4bb97' // SSH Credentials ID
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
                        sh '''
                            echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USER} --password-stdin
                            docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                        '''
                    }
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                script {
                    echo "Deploying Application on Server"
                    withCredentials([usernamePassword(credentialsId: SSH_CREDENTIALS_ID, usernameVariable: 'SSH_USER', passwordVariable: 'SSH_PASSWORD')]) {
                        // Securely handle SSH password without interpolation
                        sh """
                            sshpass -p '${SSH_PASSWORD}' ssh -o StrictHostKeyChecking=no ${SSH_USER}@${SERVER_HOST} << 'EOF'
                            echo 'Stopping and removing existing container...'
                            docker stop react-app-container || true
                            docker rm react-app-container || true

                            echo 'Pulling latest image...'
                            docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}

                            echo 'Running new container...'
                            docker run -d --name react-app-container -p 80:80 ${DOCKER_IMAGE}:${DOCKER_TAG}

                            echo 'Removing unused images...'
                            docker rmi ${DOCKER_IMAGE}:${DOCKER_TAG} || true
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
