pipeline {
    agent any
    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'v1', description: 'Docker Image Version Tag')
    }
    environment {
        DOCKER_HUB_USER = 'YOUR_DOCKERHUB_USERNAME'
        IMAGE_NAME = 'campus-energy-monitor-system'
    }
    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }
        stage('Build & Test Parallel') {
            parallel {
                stage('Build Docker Image') {
                    steps {
                        sh "docker build -t ${DOCKER_HUB_USER}/${IMAGE_NAME}:${params.IMAGE_TAG} ."
                        sh "docker tag ${DOCKER_HUB_USER}/${IMAGE_NAME}:${params.IMAGE_TAG} ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                    }
                }
                stage('Lint Code Source') {
                    steps {
                        echo 'Testing dashboard HTML structure...'
                        sh "grep -q 'Smart Campus Energy Monitoring System' index.html"
                    }
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh "echo ${PASS} | docker login -u ${USER} --password-stdin"
                    sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${params.IMAGE_TAG}"
                    sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh "sed -i 's|:${DOCKER_HUB_USER}/${IMAGE_NAME}:latest|:${DOCKER_HUB_USER}/${IMAGE_NAME}:${params.IMAGE_TAG}|g' deployment.yaml"
                sh "kubectl apply -f deployment.yaml"
                sh "kubectl rollout status deployment/campus-energy-monitor-system"
            }
        }
    }
}
