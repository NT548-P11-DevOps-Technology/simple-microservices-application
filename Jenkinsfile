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
                    sh 'sudo docker-compose build'
                    sh 'sudo docker images'
                    sh 'sudo docker-compose ps'
                }
            }
        }
        stage('Cleaning and Deploying') {
            steps {
                echo 'Cleaning...'
                sh 'sudo docker-compose down'
                sh 'sudo docker-compose ps -a'

                echo 'Deploying...'
                sh 'sudo docker-compose up'
                sh 'sudo docker-compose ps'
            }
        }
    }
    post {
        always {
            echo 'Cleaning...'
            sh 'sudo docker-compose down'
            sh 'sudo docker-compose ps -a'
            sh 'sudo docker logout'
            cleanWs()
        }
    }
}