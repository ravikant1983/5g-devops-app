pipeline {

    agent any

    environment {
        DOCKER_IMAGE = "rkg1983/5g-devops-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh '''
                    docker build \
                    -t ${DOCKER_IMAGE}:${IMAGE_TAG} \
                    .
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    docker run -d \
                    --name test-container \
                    ${DOCKER_IMAGE}:${IMAGE_TAG}

                    sleep 5

                    docker exec test-container \
                    wget -qO- http://localhost/ \
                    | grep "5G DevOps"

                    docker rm -f test-container
                '''
            }
        }

        stage('Docker Login') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASSWORD" | \
                        docker login \
                        --username "$DOCKER_USER" \
                        --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Hub') {
            steps {

                sh '''
                    docker push \
                    ${DOCKER_IMAGE}:${IMAGE_TAG}
                '''
            }
        }

        stage('Latest') {
            steps {

                sh '''
                    docker tag \
                    ${DOCKER_IMAGE}:${IMAGE_TAG} \
                    ${DOCKER_IMAGE}:latest

                    docker push \
                    ${DOCKER_IMAGE}:latest
                '''
            }
        }
    }

    post {

        always {

            sh '''
                docker logout || true
            '''
        }
    }
}
