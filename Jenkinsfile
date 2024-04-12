pipeline {
    agent any

    stages {
        stage('Authenticate with Docker Hub') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub-account', url: 'https://index.docker.io/v1/') {
                    echo 'Logged in to Docker Hub'
                }
            }
        }
        stage('Set up database') {
            steps {
                echo 'Building mysql image...'
                sh 'docker compose build mysql'
                echo 'Building phpmyadmin image...'
                sh 'docker compose build phpmyadmin'
            }
        }
        stage('Build back-end application image') {
            steps {
                echo 'student-backend is being built...'
                sh 'docker compose build student-backend'
            }
        }
        stage('Build front-end application image') {
            steps {
                echo 'student-fronend is being built...'
                sh 'docker compose build student-frontend'
            }
        }
        stage('Clean and Deploy to Dev Environment') {
            steps {
                echo 'Listing Images and containers...'
                sh 'docker images'
                sh 'docker compose ps'
            }
            steps {
                echo 'Cleaning...'
                sh 'docker compose down -v'
                sh 'echo y | docker container prune'
                sh 'docker compose ps'

                echo 'Deploying...'
                sh 'docker compose up'
                sh 'docker compose ps'
                sh 'docker network ls'
                sh 'docker volume ls'
            }
            step {
                echo 'Pushing images to Docker Hub...'
                sh 'docker compose push'
            }
        }
    }
    post {
        always {
            echo 'Cleaning...'
            sh 'docker compose down -v'
            sh 'echo y | docker container prune'
            sh 'docker compose ps'
            sh 'docker logout'
            cleanWs()
        }
    }
}