pipeline {
    agent any
    triggers { pollSCM('*/5 * * * *') 
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')

    }
    stages {
        stage('Checkout'){
            agent any
            steps{
                checkout scm
            }
        }
        stage('Init'){
            steps{
                // Permet l'authentification
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('Build'){
            steps {
                sh 'docker build -t $DOCKERHUB_CREDENTIALS_USR/reactapp:$BUILD_ID .'
            }
        }
        stage('Deliver'){
            steps {
                sh 'docker push $DOCKERHUB_CREDENTIALS_USR/reactapp:$BUILD_ID'
            }
        }
        stage('Cleanup'){
            steps {
                sh 'docker rmi $DOCKERHUB_CREDENTIALS_USR/reactapp:$BUILD_ID'
                sh 'docker logout'
            }
        }
    }
}
