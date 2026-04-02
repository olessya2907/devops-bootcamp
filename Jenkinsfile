pipeline {
    agent any

    environment {
        IMAGE_NAME      = "devops-bootcamp-backend"
        DOCKER_HUB_REPO = "psychodol/devops-bootcamp-backend"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'ls -la'
            }
        }

        stage('Build') {
            steps {
                sh """
                    docker build \
                      -t ${IMAGE_NAME}:${BUILD_NUMBER} \
                      -t ${IMAGE_NAME}:latest \
                      ./app/backend
                """
            }
        }

        stage('Test') {
            steps {
                sh """
                    docker run --rm \
                      ${IMAGE_NAME}:${BUILD_NUMBER} \
                      echo "Image OK"
                """
            }
        }

        stage('Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker tag $IMAGE_NAME:$BUILD_NUMBER $DOCKER_HUB_REPO:$BUILD_NUMBER
                        docker tag $IMAGE_NAME:latest        $DOCKER_HUB_REPO:latest
                        docker push $DOCKER_HUB_REPO:$BUILD_NUMBER
                        docker push $DOCKER_HUB_REPO:latest
                    '''
                }
            }
        }

        stage('Cleanup') {
            steps {
                sh """
                    docker rmi ${IMAGE_NAME}:${BUILD_NUMBER} || true
                    docker rmi ${IMAGE_NAME}:latest           || true
                """
            }
        }
    }

    post {
        success {
            echo "Build #${BUILD_NUMBER} pushed to Docker Hub: ${DOCKER_HUB_REPO}:${BUILD_NUMBER}"
        }
        failure {
            echo "Build #${BUILD_NUMBER} failed"
        }
    }
}
