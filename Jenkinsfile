pipeline {
    agent any

    environment {
        IMAGE_NAME = "vishalbhj/flask-two-tier-app"
        BUILD_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Pulling latest changes from GitHub..."
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh '''
                    docker build -t $IMAGE_NAME:$BUILD_TAG .
                    docker tag $IMAGE_NAME:$BUILD_TAG $IMAGE_NAME:latest
                '''
            }
        }

        stage('Push Image') {
            steps {
                echo "Pushing image to DockerHub..."
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                        echo $PASS | docker login -u $USER --password-stdin
                        docker push $IMAGE_NAME:$BUILD_TAG
                        docker push $IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Updating deployment image..."
                sh '''
                    kubectl set image deployment/flask-app flask-container=$IMAGE_NAME:$BUILD_TAG
                    kubectl rollout status deployment/flask-app
                '''
            }
        }
    }

    post {
        success {
            echo "Deployment successful. New version is live."
        }
        failure {
            echo "Build failed. Check logs."
        }
    }
}
