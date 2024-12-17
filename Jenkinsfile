pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'mynameaom/test_deploy'
        DOCKER_TAG = 'latest'
        DOCKER_HUB_USER = 'mynameaom'
        DOCKER_HUB_CREDENTIALS = 'c8ad2cd2-59b2-4f96-8980-162d28142755'
        SERVER_USER = 'root'
        SERVER_HOST = '192.168.136.134'
        SSH_CREDENTIALS_ID = '6e628bfe-ef3d-4e27-9237-2465ebf4bb97'
    }

    stages {
        stage('Deploy to Server') {
            steps {
                script {
                    echo "Deploying Application on Server"
                    withCredentials([file(credentialsId: SSH_CREDENTIALS_ID, variable: 'SSH_PRIVATE_KEY')]) {
                        sh '''
                            chmod 600 $SSH_PRIVATE_KEY
                            ssh -i $SSH_PRIVATE_KEY -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_HOST \
                            'echo "Stopping and removing existing container..."; \
                            docker stop react-app-container || true; \
                            docker rm react-app-container || true; \
                            echo "Pulling latest image..."; \
                            docker pull $DOCKER_IMAGE:$DOCKER_TAG; \
                            echo "Running new container..."; \
                            docker run -d --name react-app-container -p 80:80 $DOCKER_IMAGE:$DOCKER_TAG; \
                            echo "Removing unused images..."; \
                            docker rmi $DOCKER_IMAGE:$DOCKER_TAG || true'
                        '''
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
