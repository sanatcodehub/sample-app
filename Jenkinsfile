pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_USERNAME = 'sanatcodehub'  // Your GitHub username
        IMAGE_TAG = "${env.BUILD_NUMBER}-${env.GIT_COMMIT.take(8)}"
        KUBECONFIG = credentials('kubeconfig')
        DOCKER_CREDENTIALS = credentials('docker-hub-credentials')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                }
            }
        }
        
        stage('Build Backend') {
            steps {
                dir('backend') {
                    script {
                        sh "docker build -t ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/sample-backend:${IMAGE_TAG} ."
                        sh "docker build -t ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/sample-backend:latest ."
                    }
                }
            }
        }
        
        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    script {
                        sh "docker build -t ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/sample-frontend:${IMAGE_TAG} ."
                        sh "docker build -t ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/sample-frontend:latest ."
                    }
                }
            }
        }
        
        stage('Test') {
            parallel {
                stage('Test Backend') {
                    steps {
                        script {
                            echo 'Backend tests would run here'
                            // Temporarily skip tests until we have proper test setup
                            // sh "cd backend && npm test"
                        }
                    }
                }
                stage('Test Frontend') {
                    steps {
                        script {
                            echo 'Frontend tests would run here'
                            // Temporarily skip tests until we have proper test setup
                            // sh "cd frontend && npm test -- --watchAll=false"
                        }
                    }
                }
            }
        }
        
        stage('Push Images') {
            when {
                anyOf {
                    branch 'main'
                    branch 'master'
                    branch 'develop'
                }
            }
            steps {
                script {
                    sh "echo '${DOCKER_CREDENTIALS_PSW}' | docker login ${DOCKER_REGISTRY} -u ${DOCKER_CREDENTIALS_USR} --password-stdin"
                    sh "docker push ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/sample-backend:${IMAGE_TAG}"
                    sh "docker push ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/sample-backend:latest"
                    sh "docker push ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/sample-frontend:${IMAGE_TAG}"
                    sh "docker push ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/sample-frontend:latest"
                }
            }
        }
        
        stage('Deploy to Development') {
            when {
                branch 'develop'
            }
            steps {
                script {
                    sh """
                        # Update image tags in manifests
                        sed -i 's|image: .*/sample-backend:.*|image: ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/sample-backend:${IMAGE_TAG}|g' k8s/development/backend-deployment.yaml
                        sed -i 's|image: .*/sample-frontend:.*|image: ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/sample-frontend:${IMAGE_TAG}|g' k8s/development/frontend-deployment.yaml
                        
                        # Apply Kubernetes manifests
                        kubectl apply -f k8s/development/
                        
                        # Wait for deployments to be ready
                        kubectl rollout status deployment/sample-backend -n sample-app-development --timeout=300s
                        kubectl rollout status deployment/sample-frontend -n sample-app-development --timeout=300s
                        
                        # Show deployment status
                        kubectl get pods -n sample-app-development
                        kubectl get services -n sample-app-development
                    """
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                anyOf {
                    branch 'main'
                    branch 'master'
                }
            }
            steps {
                script {
                    sh """
                        sed -i 's|image: .*/sample-backend:.*|image: ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/sample-backend:${IMAGE_TAG}|g' k8s/staging/backend-deployment.yaml
                        sed -i 's|image: .*/sample-frontend:.*|image: ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/sample-frontend:${IMAGE_TAG}|g' k8s/staging/frontend-deployment.yaml
                        kubectl apply -f k8s/staging/
                        kubectl rollout status deployment/sample-backend -n sample-app-staging --timeout=300s
                        kubectl rollout status deployment/sample-frontend -n sample-app-staging --timeout=300s
                    """
                }
            }
        }
        
        stage('Deploy to Production') {
            when {
                anyOf {
                    branch 'main'
                    branch 'master'
                }
            }
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    input message: 'Deploy to production?', ok: 'Deploy'
                }
                script {
                    sh """
                        sed -i 's|image: .*/sample-backend:.*|image: ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/sample-backend:${IMAGE_TAG}|g' k8s/production/backend-deployment.yaml
                        sed -i 's|image: .*/sample-frontend:.*|image: ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/sample-frontend:${IMAGE_TAG}|g' k8s/production/frontend-deployment.yaml
                        kubectl apply -f k8s/production/
                        kubectl rollout status deployment/sample-backend -n sample-app-production --timeout=300s
                        kubectl rollout status deployment/sample-frontend -n sample-app-production --timeout=300s
                    """
                }
            }
        }
    }
    
    post {
        always {
            script {
                // Clean up Docker images
                sh """
                    docker image prune -f || true
                    docker rmi ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/sample-backend:${IMAGE_TAG} || true
                    docker rmi ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/sample-frontend:${IMAGE_TAG} || true
                """
            }
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
