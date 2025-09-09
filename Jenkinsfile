pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_USERNAME = 'sanat6'
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
            when { branch 'develop' }
            steps {
                script {
                    sh """
                        kubectl apply -f k8s/development/
                        kubectl rollout status deployment/sample-backend -n sample-app-development --timeout=300s
                        kubectl rollout status deployment/sample-frontend -n sample-app-development --timeout=300s
                    """
                }
            }
        }
        
        stage('Deploy to Staging') {
            when { anyOf { branch 'main'; branch 'master' } }
            steps {
                script {
                    sh """
                        kubectl apply -f k8s/staging/
                        kubectl rollout status deployment/sample-backend -n sample-app-staging --timeout=300s
                        kubectl rollout status deployment/sample-frontend -n sample-app-staging --timeout=300s
                    """
                }
            }
        }
        
        stage('Deploy to Production') {
            when { anyOf { branch 'main'; branch 'master' } }
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    input message: 'Deploy to production?', ok: 'Deploy'
                }
                script {
                    sh """
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
            sh "docker image prune -f || true"
        }
    }
}
