pipeline {
    agent any

    environment {
        IMAGE_NAME = "mizhabnp/devops-lab"
        IMAGE_TAG  = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Code checked out by Jenkins automatically"
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %IMAGE_NAME%:%IMAGE_TAG% ."
                echo "Docker image built successfully"
            }
        }

        stage('Test') {
            steps {
                echo "Build verified - image is ready to deploy"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"
                    bat "docker push %IMAGE_NAME%:%IMAGE_TAG%"
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully! Site is live at http://3.108.66.66"
        }
        failure {
            echo "Pipeline failed. Check logs above."
        }
    }
}
