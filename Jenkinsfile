pipeline {
    agent any

    environment {
        DOCKERHUB = credentials('dockerhub')
        IMAGE_NAME = "yourdockerhubusername/netflix-clone"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs --exit-code 0 --severity HIGH,CRITICAL .'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t netflix-clone .'
            }
        }

        stage('Tag & Push to DockerHub') {
            steps {
                sh """
                docker login -u ${DOCKERHUB_USR} -p ${DOCKERHUB_PSW}
                docker tag netflix-clone:latest ${IMAGE_NAME}:latest
                docker push ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --exit-code 0 --severity HIGH,CRITICAL ${IMAGE_NAME}:latest"
            }
        }

        stage('Deploy Container') {
            steps {
                sh """
                docker stop netflix || true
                docker rm netflix || true
                docker run -d --name netflix -p 80:80 ${IMAGE_NAME}:latest
                """
            }
        }
    }
}
