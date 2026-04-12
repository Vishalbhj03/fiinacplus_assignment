pipeline {
    agent any

    environment {
        IMAGE_NAME = "vishalbhj/flask-two-tier-app"
        BUILD_TAG = "${BUILD_NUMBER}"
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Pulling latest code..."
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

        stage('Test') {
            steps {
                echo "Running container for testing..."
                sh '''
                    docker run -d -p 5000:5000 --name test-container $IMAGE_NAME:$BUILD_TAG
                    sleep 5
                    curl -f http://localhost:5000/health
                '''
            }
            post {
                always {
                    echo "Cleaning test container..."
                    sh 'docker rm -f test-container || true'
                }
            }
        }

        stage('Push Image') {
            steps {
                echo "Pushing to DockerHub..."
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
                echo "Deploying to Kubernetes..."

                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh '''
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml

                        kubectl set image deployment/flask-app flask-container=$IMAGE_NAME:$BUILD_TAG
                        kubectl rollout status deployment/flask-app
                    '''
                }
            }
        }

        stage('Cleanup') {
            steps {
                echo "Cleaning unused Docker images..."
                sh 'docker system prune -f'
            }
        }
    }

    post {
        success {
            echo "Deployment successful"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}
