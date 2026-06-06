pipeline {
    agent any

    environment {
        IMAGE_NAME    = "jenkins-learning-app"
        IMAGE_TAG     = "build-${env.BUILD_NUMBER}"
        REGISTRY      = "ghcr.io"
        GITHUB_USER   = "smergon"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Building branch: ${env.BRANCH_NAME}, build #${env.BUILD_NUMBER}"
            }
        }

        stage('Build Docker image') {
            steps {
                sh "docker build -f Dockerfile.app -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
            }
        }

        stage('Run container') {
            steps {
                sh "docker run --rm ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Push to registry') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'cd6a5ea1-f1e8-472f-97ef-357e0b48e5ba',
                    usernameVariable: 'REG_USER',
                    passwordVariable: 'REG_TOKEN'
                )]) {
                    sh """
                        echo \$REG_TOKEN | docker login ${REGISTRY} -u \$REG_USER --password-stdin
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${REGISTRY}/${GITHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${REGISTRY}/${GITHUB_USER}/${IMAGE_NAME}:latest
                        docker push ${REGISTRY}/${GITHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${REGISTRY}/${GITHUB_USER}/${IMAGE_NAME}:latest
                        docker logout ${REGISTRY}
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'cd6a5ea1-f1e8-472f-97ef-357e0b48e5ba',
                    usernameVariable: 'REG_USER',
                    passwordVariable: 'REG_TOKEN'
                )]) {
                    sh """
                        echo \$REG_TOKEN | docker login ${REGISTRY} -u \$REG_USER --password-stdin
                        docker pull ${REGISTRY}/${GITHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker stop ${IMAGE_NAME} || true
                        docker rm ${IMAGE_NAME} || true
                        docker run -d --name ${IMAGE_NAME} ${REGISTRY}/${GITHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker logout ${REGISTRY}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployed ${IMAGE_NAME}:${IMAGE_TAG} successfully"
        }
        failure {
            echo "Pipeline failed — check console output above"
        }
        always {
            sh "docker image prune -f"
            echo "Build #${env.BUILD_NUMBER} complete"
        }
    }
}
