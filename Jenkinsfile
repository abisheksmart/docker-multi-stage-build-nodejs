pipeline {
    agent any

    environment {
        IMAGE_NAME = "myimage"
        CONTAINER_NAME = "myimage:v1"
    }

    stages {
        stage('Checkout') {
            steps {

                git credentialsId: '0b9d36ce-b24d-4222-a90d-077955446142', url: 'https://github.com/abisheksmart/docker-multi-stage-build-nodejs.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $IMAGE_NAME .'
                }
            }
        }

        stage('Run Lint & Tests') {
            steps {
                script {
                    sh '''
                    docker build --target TEST -t ${IMAGE_NAME}-test .
                    docker run --rm ${IMAGE_NAME}-test
                    '''
                }
            }
        }

        stage('Run App') {
            steps {
                script {
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
    }

    post {
        always {
            echo 'Pipeline completed.'
        }
    }
}
