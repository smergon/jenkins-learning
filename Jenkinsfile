pipeline {
	agent any

	triggers {
        pollSCM('H/10 * * * *')
    }

	stages {
		stage('Checkout') {
			steps {
				echo "Checked out branch: ${env.BRANCH_NAME}"
			}
		}
		stage('Build') {
			steps {
				sh 'echo "Building.."'
				sh './app.sh'
			}
		}
		stage('Test') {
			steps {
				sh 'echo "Running tests..."'
			}
		}
		stage('Docker Check') {
			steps {
				sh 'docker --version'
			}
		}
	}

	post {
		success {
			echo 'Pipeline completed'
		}
		failure {
			echo 'Pipeline failed'
		}
	}
}
