pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_USERNAME = 'sanatcodehub'
        IMAGE_TAG = "${env.BUILD_NUMBER}-${env.GIT_COMMIT.take(8)}"
        KUBECONFIG = credentials('kubeconfig')
        DOCKER_CREDENTIALS = credentials('dockerhub-creds')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    // Store these for post section
                    env.DOCKER_REG = env.DOCKER_REGISTRY
                    env.DOCKER_USER = env.DOCKER_USERNAME
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
                            // sh "cd backend && npm test"
                        }
                    }
                }
                stage('Test Frontend') {
                    steps {
                        script {
                            echo 'Frontend tests would run here'
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
                        echo "Deploying to development environment..."
                        sed -i 's|image: .*/sample-backend:.*|image: ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/sample-backend:${IMAGE_TAG}|g' k8s/development/backend-deployment.yaml
                        sed -i 's|image: .*/sample-frontend:.*|image: ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/sample-frontend:${IMAGE_TAG}|g' k8s/development/frontend-deployment.yaml
                        kubectl apply -f k8s/development/
                        kubectl rollout status deployment/sample-backend -n sample-app-development --timeout=300s || true
                        kubectl rollout status deployment/sample-frontend -n sample-app-development --timeout=300s || true
                        kubectl get pods -n sample-app-development || true
                        kubectl get services -n sample-app-development || true
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
                        echo "Deploying to staging environment..."
                        sed -i 's|image: .*/sample-backend:.*|image: ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/sample-backend:${IMAGE_TAG}|g' k8s/staging/backend-deployment.yaml
                        sed -i 's|image: .*/sample-frontend:.*|image: ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/sample-frontend:${IMAGE_TAG}|g' k8s/staging/frontend-deployment.yaml
                        kubectl apply -f k8s/staging/ || true
                        kubectl rollout status deployment/sample-backend -n sample-app-staging --timeout=300s || true
                        kubectl rollout status deployment/sample-frontend -n sample-app-staging --timeout=300s || true
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
                        echo "Deploying to production environment..."
                        sed -i 's|image: .*/sample-backend:.*|image: ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/sample-backend:${IMAGE_TAG}|g' k8s/production/backend-deployment.yaml
                        sed -i 's|image: .*/sample-frontend:.*|image: ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/sample-frontend:${IMAGE_TAG}|g' k8s/production/frontend-deployment.yaml
                        kubectl apply -f k8s/production/ || true
                        kubectl rollout status deployment/sample-backend -n sample-app-production --timeout=300s || true
                        kubectl rollout status deployment/sample-frontend -n sample-app-production --timeout=300s || true
                    """
                }
            }
        }
    }
    
    post {
        always {
            script {
                try {
                    // Use environment variables stored during checkout
                    def dockerReg = env.DOCKER_REG ?: 'docker.io'
                    def dockerUser = env.DOCKER_USER ?: 'sanatcodehub'
                    def imageTag = env.IMAGE_TAG ?: 'latest'
                    
                    echo "Cleaning up Docker images..."
                    sh """
                        docker image prune -f || true
                        docker rmi ${dockerReg}/${dockerUser}/sample-backend:${imageTag} || true
                        docker rmi ${dockerReg}/${dockerUser}/sample-frontend:${imageTag} || true
                        docker rmi ${dockerReg}/${dockerUser}/sample-backend:latest || true
                        docker rmi ${dockerReg}/${dockerUser}/sample-frontend:latest || true
                    """
                } catch (Exception e) {
                    echo "Error during cleanup: ${e.getMessage()}"
                }
            }
        }
        success {
            echo 'Pipeline completed successfully! 🎉'
        }
        failure {
            echo 'Pipeline failed! ❌'
        }
    }
}
