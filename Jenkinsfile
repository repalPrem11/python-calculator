pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'dockerhub-creds' // Jenkins credential ID
        DOCKER_HUB_REPO = 'premrepal/python-calculator' // Your Docker Hub repo
        IMAGE_TAG = "latest" // or you can use "${env.BUILD_NUMBER}" for unique tags
    }

    stages {
        stage('Check Environment') {
            steps {
                sh 'java -version'
                sh 'docker --version'
                sh 'git --version'
            }
        }

        stage('Clone Repo') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/repalPrem11/python-calculator.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_HUB_REPO}:${IMAGE_TAG} ."
            }
        }

        stage('List Images') {
            steps {
                sh "docker images | grep python-calculator"
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Login and push using credentials stored in Jenkins
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_HUB_CREDENTIALS}") {
                        docker.image("${DOCKER_HUB_REPO}:${IMAGE_TAG}").push()
                        docker.image("${DOCKER_HUB_REPO}:${IMAGE_TAG}").push('latest') // optional
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Docker image built and pushed successfully!"
        }
        failure {
            echo "Build or push failed."
        }
    }
}
