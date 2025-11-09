pipeline {
    agent any
    triggers { pollSCM('*/5 * * * *') }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
    }
    stages {
        stage('Checkout') {
            agent any
            steps{
                echo '========== Starting Checkout Stage =========='
                checkout scm
                echo '========== Checkout Completed Successfully =========='
            }
        }

        stage('Docker Login') {
            steps {
                echo '========== Starting Docker Login =========='
                echo "Logging in as user: ${DOCKERHUB_CREDENTIALS_USR}"
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                echo '========== Docker Login Successful =========='
            }
        }

        stage('Build & Push Docker Compose Images') {
            steps {
                echo '========== Starting Docker Compose Build =========='
                sh '''
                    docker compose -f docker-compose.yml build
                '''
                echo '========== Build Completed Successfully =========='
                
                echo '========== Starting Docker Compose Push =========='
                sh '''
                    docker compose -f docker-compose.yml push
                '''
                echo '========== Push Completed Successfully =========='
            }
        }

        stage('Cleanup') {
            steps {
                echo '========== Starting Cleanup Stage =========='
                echo 'Removing Docker Compose resources...'
                sh 'docker compose -f docker-compose.yml down --rmi all || true'
                echo 'Docker Compose cleanup completed'
                
                echo 'Logging out from Docker...'
                sh 'docker logout'
                echo '========== Cleanup Completed Successfully =========='
            }
        }
    }
}