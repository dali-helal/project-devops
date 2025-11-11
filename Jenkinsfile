pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        DOCKERHUB_USERNAME   = 'dali917'

        BUILD_TAG     = "${env.BUILD_NUMBER}"
        SERVER_IMAGE  = "${DOCKERHUB_USERNAME}/mern-server"
        CLIENT_IMAGE  = "${DOCKERHUB_USERNAME}/mern-client"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "========== GIT CHECKOUT =========="
                checkout scm
            }
        }

        stage('Build Docker Images') {
            steps {
                echo "========== DOCKER BUILD =========="
                sh """
                    docker build -t ${SERVER_IMAGE}:${BUILD_TAG} -f server/Dockerfile ./server
                    docker build -t ${CLIENT_IMAGE}:${BUILD_TAG} -f client/Dockerfile ./client
                """
            }
        }

        stage('Docker Login & Push') {
            steps {
                echo "========== DOCKER PUSH =========="
                sh """
                    echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin
                    docker push ${SERVER_IMAGE}:${BUILD_TAG}
                    docker push ${CLIENT_IMAGE}:${BUILD_TAG}
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "========== DEPLOY TO K8s =========="

                sh """
                docker run --rm \
                -v /root/.kube:/root/.kube \
                -v /var/jenkins_home/workspace/mini-project-devops/k8s:/k8s \
                --entrypoint /bin/sh \
                bitnami/kubectl:latest -c "
                
                        echo '--- Updating server image ---'
                        kubectl set image deployment/mern-server mern-server=${SERVER_IMAGE}:${BUILD_TAG} --record || true
                        kubectl apply -f /k8s/deployment-server.yaml
                        kubectl rollout status deployment/mern-server

                        echo '--- Updating client image ---'
                        kubectl set image deployment/mern-client mern-client=${CLIENT_IMAGE}:${BUILD_TAG} --record || true
                        kubectl apply -f /k8s/deployment-client.yaml
                        kubectl rollout status deployment/mern-client
                    "
                """
            }
        }
    }

    post {
        always {
            echo "========== CLEANUP =========="
            sh """
                docker logout || true
                docker system prune -f || true
            """
        }

        failure {
            echo """
=========================================
              PIPELINE FAILED
=========================================
Build Number: ${BUILD_TAG}
Check logs for details
=========================================
"""
        }

        success {
            echo """
=========================================
         DEPLOYMENT SUCCESS ✅
-----------------------------------------
Build Number: ${BUILD_TAG}
=========================================
"""
        }
    }
}
