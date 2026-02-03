pipeline {
    agent any

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
                sh 'docker build -t python-calculator:latest .'
            }
        }

        stage('List Images') {
            steps {
                sh 'docker images | grep python-calculator'
            }
        }
    }
}


