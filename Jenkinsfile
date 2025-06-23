pipeline {
    agent any

    environment {
        IMAGE_NAME = "myimage"
        CONTAINER_NAME = "myimage:v1"
    }

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: '0b9d36ce-b24d-4222-a90d-077955446142', 
                    url: 'https://github.com/abisheksmart/docker-multi-stage-build-nodejs.git'
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
                sh 'DOCKER_BUILDKIT=1 docker build -t $IMAGE_NAME .'
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
                docker network create --driver bridge my-custom-network || true
                docker volume create flask-logs || true

                docker run -d --rm \
                    --name $CONTAINER_NAME \
                    --network my-custom-network \
                    -v flask-logs:/app/logs \
                    -p 1337:1337 \
                    $IMAGE_NAME
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

