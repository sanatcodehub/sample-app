# Sample App - Jenkins CI/CD Pipeline Demo

This is a complete full-stack application with automated CI/CD pipeline using Jenkins, Docker, and Kubernetes.

## Quick Start

1. Start minikube: `minikube start`
2. Build locally: `docker-compose up --build`
3. Access at: http://localhost:3000

## Architecture

- **Frontend**: React.js with Tailwind CSS
- **Backend**: Node.js/Express REST API
- **Database**: In-memory (for demo)
- **Container**: Docker
- **Orchestration**: Kubernetes
- **CI/CD**: Jenkins Pipeline

## Environments

- Development: http://<minikube-ip>:30000
- Staging: http://<minikube-ip>:30001  
- Production: http://<minikube-ip>:30002

## Created by setup script with Docker Hub user: sanat6
test
