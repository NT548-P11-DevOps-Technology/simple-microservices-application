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
                echo 'Setting up databases...'
                sh '''
                    docker compose -f docker-compose.yml build student-service-mysql
                    docker compose -f docker-compose.yml build auth-service-mysql
                    docker compose -f docker-compose.yml build mongodb
                    docker compose -f docker-compose.yml build postgresql
                '''
            }
        }
        stage('Set up admin tools') {
            steps {
                echo 'Setting up admin tools...'
                sh '''
                    docker compose -f docker-compose.yml build auth-phpmyadmin
                    docker compose -f docker-compose.yml build student-phpmyadmin
                    docker compose -f docker-compose.yml build mongo-express
                    docker compose -f docker-compose.yml build pgadmin
                '''
            }
        }
        stage('Build back-end application images') {
            steps {
                echo 'Building back-end application images...'
                sh '''
                    cp class-management-auth-service/Dockerfile .
                    docker compose -f docker-compose.yml build class-mangement-auth-service
                    cp class-management-student-service/Dockerfile .
                    docker compose -f docker-compose.yml build class-management-student-service
                    cp class-management-lecturer-service/Dockerfile .
                    docker compose -f docker-compose.yml build class-management-lecturer-service
                    cp class-management-class-service/Dockerfile .
                    docker compose -f docker-compose.yml build class-management-class-service
                '''
            }
        }
        stage('Build front-end application image') {
            steps {
                echo 'Building front-end application image...'
                sh '''
                    cp class-management-fe/Dockerfile .
                    docker compose -f docker-compose.yml build class-mangement-fe
                '''
            }
        }
        // stage('Scan images with Trivy') {
        //     steps {
        //         sh '''
        //             # Ensure the file exists and is writable
        //             touch ${WORKSPACE}/trivy-report.txt
        //             chmod +w ${WORKSPACE}/trivy-report.txt

        //             # Remove the file if it exists
        //             if [ -f ${WORKSPACE}/trivy-report.txt ]; then
        //                 rm ${WORKSPACE}/trivy-report.txt
        //             fi
        //             trivy image --severity HIGH,CRITICAL chucthien03/class-management-auth-service >> ${WORKSPACE}/trivy-report.txt
        //             trivy image --severity HIGH,CRITICAL chucthien03/class-management-student-service >> ${WORKSPACE}/trivy-report.txt
        //             trivy image --severity HIGH,CRITICAL chucthien03/class-management-lecturer-service >> ${WORKSPACE}/trivy-report.txt
        //             trivy image --severity HIGH,CRITICAL chucthien03/class-management-class-service >> ${WORKSPACE}/trivy-report.txt
        //             trivy image --severity HIGH,CRITICAL chucthien03/class-management-fe >> ${WORKSPACE}/trivy-report.txt
        //         '''
        //     }
        // }
        stage('Push images to Docker Hub') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub-account', url: 'https://index.docker.io/v1/') {
                    echo 'Pushing images to Docker Hub...'
                    sh 'docker compose push'
                }
            }
        }
        stage('Clean and Deploy to Dev Environment') {
            steps {
                echo 'Listing images and containers...'
                sh 'docker images'
                sh 'docker compose ps'

                echo 'Cleaning...'
                sh 'docker rmi $(docker images -f "dangling=true" -q)'
                sh 'docker compose down -v'
                sh 'echo y | docker container prune'
                sh 'docker compose ps'

                echo 'Deploying to Dev Environment...'
                sh 'docker compose up -d'
                sh 'docker compose ps'
                sh 'docker network ls'
                sh 'docker volume ls'
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-token', namespace: 'jenkins', restrictKubeConfigAccess: false, serverUrl: 'https://ec2-52-221-253-100.ap-southeast-1.compute.amazonaws.com:6443') {
                    echo 'Deploying to Kubernetes...'
                    sh 'kubectl apply -f secrets.yaml'
                    sh 'kubectl apply -f configMap.yaml'
                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
                    echo 'Checking deployment status...'
                    sh 'kubectl get svc'
                    sh 'kubectl get pods'
                }
            }
        }
    }
    post {
        always {
            echo 'Cleaning...'
            sh 'docker logout'
            cleanWs()
        }
        success {
            echo 'Deployment to Dev Environment is successful!'
        }
        failure {
            echo 'Deployment to Dev Environment failed!'
        }
        unstable {
            echo 'Deployment to Dev Environment is unstable!'
        }
        changed {
            echo 'Deployment to Dev Environment is changed!'
        }
    }
}
