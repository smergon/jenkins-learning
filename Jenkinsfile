pipeline {
	agent any

	environment {
        IMAGE_NAME = "jenkins-learning-app"
        IMAGE_TAG  = "build-${env.BUILD_NUMBER}"
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
		stage('Verify image exists') {
            steps {
                sh "docker images ${IMAGE_NAME}"
            }
        }
	}

	post {
		success {
            echo "Image ${IMAGE_NAME}:${IMAGE_TAG} built successfully"
        }
        failure {
            echo "Build failed — check console output above"
        }
        always {
            echo "Build #${env.BUILD_NUMBER} complete"
        }
	}
}
