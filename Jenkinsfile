pipeline {
    agent any
    
    triggers { 
        pollSCM('*/5 * * * *') 
    }
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        DOCKERHUB_USERNAME = 'dali917'
        BUILD_TAG = "${env.BUILD_NUMBER}"
        SERVER_IMAGE = "${DOCKERHUB_USERNAME}/mern-server"
        CLIENT_IMAGE = "${DOCKERHUB_USERNAME}/mern-client"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '=========================================='
                echo '========== Starting Checkout =========='
                echo '=========================================='
                checkout scm
                echo 'Checkout Completed Successfully'
            }
        }

        stage('Docker Login') {
            steps {
                echo '=========================================='
                echo '========== Docker Login =========='
                echo '=========================================='
                echo "Docker Hub Username: ${DOCKERHUB_USERNAME}"
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                echo 'Docker Login Successful'
            }
        }

        stage('Build Docker Images') {
            steps {
                echo '=========================================='
                echo '========== Building MERN Stack Images =========='
                echo '=========================================='
                echo "Build Number: ${BUILD_TAG}"
                echo "Building Server and Client images..."
                
                sh """
                    export BUILD_NUMBER=${BUILD_TAG}
                    docker compose -f docker-compose.yml build server client
                """
                
                echo 'Build Completed Successfully'
                echo ''
                echo 'Built Images:'
                sh """
                    docker images | grep '${DOCKERHUB_USERNAME}' | grep -E 'mern-server|mern-client'
                """
            }
        }

        stage('Tag Images as Latest') {
            steps {
                echo '=========================================='
                echo '========== Tagging Images =========='
                echo '=========================================='
                
                sh """
                    docker tag ${SERVER_IMAGE}:${BUILD_TAG} ${SERVER_IMAGE}:latest
                    echo "Tagged ${SERVER_IMAGE}:${BUILD_TAG} as latest"
                    
                    docker tag ${CLIENT_IMAGE}:${BUILD_TAG} ${CLIENT_IMAGE}:latest
                    echo "Tagged ${CLIENT_IMAGE}:${BUILD_TAG} as latest"
                """
                
                echo ''
                echo 'Current Images with Tags:'
                sh """
                    docker images | grep '${DOCKERHUB_USERNAME}' | grep -E 'mern-server|mern-client'
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo '=========================================='
                echo '========== Pushing to Docker Hub =========='
                echo '=========================================='
                
                sh """
                    echo "Pushing server image..."
                    docker push ${SERVER_IMAGE}:${BUILD_TAG}
                    docker push ${SERVER_IMAGE}:latest
                    echo "Server image pushed"
                    
                    echo ""
                    echo "Pushing client image..."
                    docker push ${CLIENT_IMAGE}:${BUILD_TAG}
                    docker push ${CLIENT_IMAGE}:latest
                    echo "Client image pushed"
                """
                
                echo ''
                echo '=========================================='
                echo 'All Images Pushed Successfully!'
                echo '=========================================='
                echo ''
                echo ' Pushed Images:'
                echo "   - ${SERVER_IMAGE}:${BUILD_TAG}"
                echo "   - ${SERVER_IMAGE}:latest"
                echo "   - ${CLIENT_IMAGE}:${BUILD_TAG}"
                echo "   - ${CLIENT_IMAGE}:latest"
                echo ''
                echo " View your repositories:"
                echo "   Server: https://hub.docker.com/r/${DOCKERHUB_USERNAME}/mern-server/tags"
                echo "   Client: https://hub.docker.com/r/${DOCKERHUB_USERNAME}/mern-client/tags"
            }
        }

        stage('Cleanup Old Images') {
            steps {
                echo '=========================================='
                echo '========== Cleaning Up Old Images =========='
                echo '=========================================='
                
                sh """
                    echo " Images before cleanup:"
                    docker images | grep '${DOCKERHUB_USERNAME}' | grep -E 'mern-server|mern-client' || echo "No images found"
                    echo ""
                    
                    echo " Removing old builds (keeping latest and build #${BUILD_TAG})..."
                    
                    # Remove old server images
                    for tag in \$(docker images ${SERVER_IMAGE} --format "{{.Tag}}" | grep -v 'latest' | grep -v '${BUILD_TAG}'); do
                        echo "  Removing ${SERVER_IMAGE}:\$tag"
                        docker rmi ${SERVER_IMAGE}:\$tag || true
                    done
                    
                    # Remove old client images
                    for tag in \$(docker images ${CLIENT_IMAGE} --format "{{.Tag}}" | grep -v 'latest' | grep -v '${BUILD_TAG}'); do
                        echo "  Removing ${CLIENT_IMAGE}:\$tag"
                        docker rmi ${CLIENT_IMAGE}:\$tag || true
                    done
                    
                    # Remove dangling images
                    docker image prune -f
                    
                    echo ""
                    echo " Images after cleanup:"
                    docker images | grep '${DOCKERHUB_USERNAME}' | grep -E 'mern-server|mern-client' || echo "No images found"
                """
                
                echo ' Cleanup Completed'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo '=========================================='
                echo '========== Deploy to Kubernetes =========='
                echo '=========================================='
                
                script {
                    sh """
                        kubectl set image deployment/mern-server mern-server=${SERVER_IMAGE}:${BUILD_TAG} --record || \
                        kubectl apply -f k8s/deployment-server.yaml
                        kubectl rollout status deployment/mern-server

                        kubectl set image deployment/mern-client mern-client=${CLIENT_IMAGE}:${BUILD_TAG} --record || \
                        kubectl apply -f k8s/deployment-client.yaml
                        kubectl rollout status deployment/mern-client
                    """
                }
                
                echo 'Kubernetes Deployment Completed'
            }
        }

        stage('Docker Logout') {
            steps {
                echo '=========================================='
                echo '========== Docker Logout =========='
                echo '=========================================='
                sh 'docker logout'
                echo ' Logged out from Docker Hub'
            }
        }
    }
    
    post {
        success {
            echo ''
            echo '=========================================='
            echo '  PIPELINE COMPLETED SUCCESSFULLY '
            echo ''
            echo "   Build Number: ${BUILD_TAG}"
            echo "   Tags Created: ${BUILD_TAG}, latest"
            echo ''
            echo "  Docker Hub Repositories:"
            echo "     Server: https://hub.docker.com/r/${DOCKERHUB_USERNAME}/mern-server"
            echo "     Client: https://hub.docker.com/r/${DOCKERHUB_USERNAME}/mern-client"
            echo ''
        }
        
        failure {
            echo ''
            echo '=========================================='
            echo '  PIPELINE FAILED'
            echo ''
            echo "  Build Number: ${BUILD_TAG}"
            echo "  Check the logs above for error details"
            echo ''
            echo '=========================================='
        }
        
        always {
            echo 'Performing final cleanup...'
            sh '''
                docker logout || true
                docker system prune -f || true
            '''
            echo 'Final cleanup completed'
        }
    }
}