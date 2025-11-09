pipeline {
    agent any
    triggers { pollSCM('*/5 * * * *') }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/dali-helal/project-devops.git',
                    credentialsId: 'github-token'
            }
        }

        stage('Docker Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Build & Push Docker Compose Images') {
            steps {
                sh '''
                    docker-compose -f docker-compose.yml build
                    docker-compose -f docker-compose.yml push
                '''
            }
        }

        stage('Cleanup') {
            steps {
                sh 'docker-compose -f docker-compose.yml down --rmi all || true'
                sh 'docker logout'
            }
        }
    }
}
