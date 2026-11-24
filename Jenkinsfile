pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'ksi2027'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build server image') {
            steps {
                bat 'docker build -t %DOCKERHUB_USER%/mern-server:latest ./Server'
            }
        }

        stage('Build client image') {
            steps {
                bat 'docker build -t %DOCKERHUB_USER%/mern-client:latest ./Client'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub',
                                 usernameVariable: 'DH_USER',
                                 passwordVariable: 'DH_PASS')]) {
                    bat 'echo %DH_PASS%| docker login -u %DH_USER% --password-stdin'
                    bat 'docker push %DOCKERHUB_USER%/mern-server:latest'
                    bat 'docker push %DOCKERHUB_USER%/mern-client:latest'
                }
            }
        }

        stage('Run containers') {
            steps {
                bat 'docker compose up -d'
                bat 'docker ps'
            }
        }
    }

    post {
        always {
            bat 'docker logout'
        }
    }
}