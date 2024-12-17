pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'mynameaom/test_deploy'  // ชื่อ image บน Docker Hub
        DOCKER_TAG = 'latest'
        DOCKER_HUB_USER = 'mynameaom'  // Username ของ Docker Hub
        DOCKER_HUB_PASSWORD = credentials('Access_Docker_Hub_Aom') // Credentials ID ใน Jenkins
        SERVER_USER = 'superadmin'
        SERVER_HOST = '192.168.136.134'
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
                    sh """
                        echo ${DOCKER_HUB_PASSWORD} | docker login -u ${DOCKER_HUB_USER} --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                script {
                    echo "Deploying Application on Server"
                    sh """
                        ssh ${SERVER_USER}@${SERVER_HOST} "
                            echo 'Stopping and removing existing container...'
                            docker stop react-app-container || true
                            docker rm react-app-container || true

                            echo 'Pulling latest image...'
                            docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}

                            echo 'Running new container...'
                            docker run -d --name react-app-container -p 80:80 ${DOCKER_IMAGE}:${DOCKER_TAG}

                            echo 'Removing unused images...'
                            docker rmi ${DOCKER_IMAGE}:${DOCKER_TAG} || true
                        "
                    """
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
