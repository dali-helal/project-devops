pipeline {
    agent any
    triggers { pollSCM('*/5 * * * *') }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
    }
    stages {
        stage('Checkout'){
            steps{
                checkout scm
            }
        }
        stage('Init'){
            steps{
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('Build'){
            steps {
                sh 'docker build -t $DOCKERHUB_CREDENTIALS_USR/reactapp:$BUILD_NUMBER .'
            }
        }
        stage('Deliver'){
            steps {
                sh 'docker push $DOCKERHUB_CREDENTIALS_USR/reactapp:$BUILD_NUMBER'
            }
        }
        stage('Cleanup'){
            steps {
                sh 'docker rmi $DOCKERHUB_CREDENTIALS_USR/reactapp:$BUILD_NUMBER || true'
                sh 'docker logout'
            }
        }
    }
}
