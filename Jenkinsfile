pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'kasimbasaragi'
        IMAGE_NAME = 'system-health-monitoring'
        APP_NAME = 'system-health-monitoring'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        DOCKER_IMAGE = "${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh """
                        echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
                        docker push ${DOCKER_IMAGE}
                    """
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh """
                        export APP_NAME=${APP_NAME}
                        export DOCKER_IMAGE=${DOCKER_IMAGE}
                        export KUBECONFIG=${KUBECONFIG}
                        envsubst < k8s/deployment.yaml | kubectl apply -f -
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment successful! App: ${APP_NAME}, Image: ${DOCKER_IMAGE}"
        }
        failure {
            echo "Deployment failed."
        }
    }
}
