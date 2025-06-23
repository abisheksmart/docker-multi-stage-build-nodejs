pipeline {
    agent any

    environment {
        IMAGE_NAME = "myimage"
        IMAGE_TAG = "v1"
        CONTAINER_NAME = "myimage-container"
    }

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: '0b9d36ce-b24d-4222-a90d-077955446142',
                    url: 'https://github.com/abisheksmart/docker-multi-stage-build-nodejs.git'
            }
        }

        stage('Debug Vars') {
            steps {
                sh '''
                echo "IMAGE_NAME=$IMAGE_NAME"
                echo "IMAGE_TAG=$IMAGE_TAG"
                '''
            }
        }

        stage('Install Dependencies & Lint') {
            steps {
                sh '''
                yarn install
                yarn run lint
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                echo "Building Docker image with tag $IMAGE_NAME:$IMAGE_TAG"
                docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Run Tests in Container') {
            steps {
                sh '''
                docker build --target TEST -t ${IMAGE_NAME}-test .
                docker run --rm ${IMAGE_NAME}-test
                '''
            }
        }

        stage('Run App') {
            steps {
                sh '''
                docker network inspect my-custom-network >/dev/null 2>&1 || docker network create --driver bridge my-custom-network
                docker volume inspect flask-logs >/dev/null 2>&1 || docker volume create flask-logs

                docker run -d --rm \
                    --name $CONTAINER_NAME \
                    --network my-custom-network \
                    -v flask-logs:/app/logs \
                    -p 1337:1337 \
                    $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed. Cleaning up unused Docker resources...'
            sh 'docker system prune -f --volumes'
        }
    }
}
