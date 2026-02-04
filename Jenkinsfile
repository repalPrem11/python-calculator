pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'dockerhub-creds'          // Jenkins credential ID
        DOCKER_HUB_REPO = 'premrepal/python-calculator'     // Your Docker Hub repo
        IMAGE_TAG = "${BUILD_NUMBER}"                        // Unique image tag
        GIT_REPO = 'https://github.com/repalPrem11/python-calculator.git'
        GIT_BRANCH = 'main'
        GIT_CREDENTIALS = 'github-creds'                    // Jenkins credentials ID for Git (username + PAT)
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
                git branch: "${GIT_BRANCH}",
                    url: "${GIT_REPO}",
                    credentialsId: "${GIT_CREDENTIALS}"
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
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_HUB_CREDENTIALS}") {
                        docker.image("${DOCKER_HUB_REPO}:${IMAGE_TAG}").push()
                        docker.image("${DOCKER_HUB_REPO}:${IMAGE_TAG}").push('latest') // optional
                    }
                }
            }
        }

        stage('Update Deployment Manifest in Git') {
            steps {
                script {
                    // Inject Git username + PAT from Jenkins credentials
                    withCredentials([usernamePassword(credentialsId: "${GIT_CREDENTIALS}", usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                        sh """
                        cd k8s
                        # Update the Deployment YAML with the new image tag
                        sed -i "s|image: ${DOCKER_HUB_REPO}:.*|image: ${DOCKER_HUB_REPO}:${IMAGE_TAG}|" deployment.yaml
                        git add deployment.yaml
                        git config user.email "jenkins@example.com"
                        git config user.name "Jenkins CI"
                        git commit -m "Update python-calculator image to build ${IMAGE_TAG}"
                        git push https://${GIT_USER}:${GIT_TOKEN}@github.com/repalPrem11/python-calculator.git ${GIT_BRANCH}
                        """
                    }
                }
            }
        }

        stage('Cleanup Old Docker Images') {
            steps {
                script {
                    // Optional: Remove local old images to save space
                    sh "docker images | grep ${DOCKER_HUB_REPO} | awk '{print \$3}' | xargs -r docker rmi -f"
                }
            }
        }
    }

    post {
        success {
            echo "Docker image built, pushed, manifest updated, and old images cleaned successfully!"
        }
        failure {
            echo "Build, push, or Git update failed."
        }
    }
}
