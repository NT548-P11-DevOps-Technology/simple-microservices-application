pipeline {
    agent any

    stages {
        stage('Check out') {
            steps {
                echo 'Checking out code...'
                checkout scm
            }
        }
        stage('Build') {
            steps {
                echo 'Authenticating with Docker Hub...'
                withDockerRegistry(credentialsId: 'dockerhub-account', url: 'https://index.docker.io/v1/') {
                    echo 'Building Docker image...'
                    sh 'docker compose build'
                    sh 'docker images'
                    sh 'docker compose ps'
                }               
            }
        }
        stage('Cleaning and Deploying') {
            steps {
                echo 'Cleaning...'
                sh 'docker compose down'
                sh 'docker compose ps'

                echo 'Deploying...'
                sh 'docker compose up'
                sh 'docker compose ps'
                sh 'docker network ls'
                sh 'docker volume ls'
            }
        }
    }
    post {
        always {
            echo 'Cleaning...'
            sh 'docker compose down'
            sh 'docker compose ps'
            sh 'docker logout'
            cleanWs()
        }
    }
}