pipeline {
    agent any

    environment {
        REGISTRY = "ramana44"
        IMAGE_NAME = "simple-webapp"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t $REGISTRY/$IMAGE_NAME:$TAG .
                """
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh """
                docker push $REGISTRY/$IMAGE_NAME:$TAG
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                kubectl set image deployment/simple-deployment \
                simple-container=$REGISTRY/$IMAGE_NAME:$TAG --record

                kubectl rollout status deployment/simple-deployment
                """
            }
        }
    }

    post {
        always {
            echo "Pipeline complete"
        }
    }
}
