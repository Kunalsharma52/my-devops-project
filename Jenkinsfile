pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'kunalsh52'
        IMAGE_NAME = 'my-devops-app'
        CONTAINER_NAME = 'my-devops-container'
        HOST_PORT = '8081'
        CONTAINER_PORT = '80'
    }

    stages {

        stage('1. Pull Code from GitHub') {
            steps {
                echo 'Downloading application source code...'

                checkout scm

                echo 'GitHub checkout completed!'
            }
        }

        stage('2. Build Docker Image') {
            steps {
                echo 'Building Docker image...'

                sh '''
                    docker build \
                        -t "$DOCKERHUB_USER/$IMAGE_NAME:$BUILD_NUMBER" \
                        -t "$DOCKERHUB_USER/$IMAGE_NAME:latest" .
                '''

                echo 'Docker image built successfully!'
            }
        }

        stage('3. Push Image to Docker Hub') {
            steps {
                echo 'Logging in to Docker Hub and pushing image...'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        set +x
                        set -eu

                        echo "$DOCKER_PASS" | docker login \
                            --username "$DOCKER_USER" \
                            --password-stdin

                        docker push "$DOCKERHUB_USER/$IMAGE_NAME:$BUILD_NUMBER"

                        docker push "$DOCKERHUB_USER/$IMAGE_NAME:latest"

                        docker logout
                    '''
                }

                echo 'Image pushed to Docker Hub!'
            }
        }

        stage('4. Deploy Application Locally') {
            steps {
                echo 'Pulling image and deploying application...'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        set +x
                        set -eu

                        echo "$DOCKER_PASS" | docker login \
                            --username "$DOCKER_USER" \
                            --password-stdin

                        docker pull "$DOCKERHUB_USER/$IMAGE_NAME:$BUILD_NUMBER"

                        docker logout

                        docker rm -f "$CONTAINER_NAME" || true

                        docker run -d \
                            --name "$CONTAINER_NAME" \
                            --restart unless-stopped \
                            -p "$HOST_PORT:$CONTAINER_PORT" \
                            "$DOCKERHUB_USER/$IMAGE_NAME:$BUILD_NUMBER"
                    '''
                }

                echo 'Application deployed successfully!'
            }
        }
    }

    post {
        success {
            echo '========================================'
            echo 'CI/CD PIPELINE COMPLETED SUCCESSFULLY!'
            echo '========================================'
            echo 'Application URL: http://localhost:8081'
        }

        failure {
            echo 'CI/CD pipeline failed. Check Console Output.'
        }
    }
}
