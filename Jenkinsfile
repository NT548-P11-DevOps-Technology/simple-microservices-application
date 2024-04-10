pipeline {
    agent any

    stages {
        stage('Check out') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                withDockerRegistry(credentialsId: 'docker-account', url: 'https://index.docker.io/v1/') {
                    sh 'docker-compose build'
                    sh 'docker images'
                    sh 'docker-compose ps'
                }
            }
        }
        stage('Cleaning and Deploying') {
            steps {
                echo 'Cleaning...'
                sh 'docker-compose down'
                sh 'docker-compose ps -a'

                echo 'Deploying...'
                sh 'docker-compose up'
                sh 'docker-compose ps'
            }
        }
    }
    post {
        always {
            echo 'Cleaning...'
            sh 'docker-compose down'
            sh 'docker-compose ps -a'
            sh 'docker logout'
            cleanWs()
        }
    }
}