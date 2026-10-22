pipeline {
    agent any
    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'v1', description: 'Docker Image Version Tag')
    }
    environment {
        DOCKER_HUB_USER = '78619'
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
                        bat "docker build -t %DOCKER_HUB_USER%/%IMAGE_NAME%:%IMAGE_TAG% ."
                        bat "docker tag %DOCKER_HUB_USER%/%IMAGE_NAME%:%IMAGE_TAG% %DOCKER_HUB_USER%/%IMAGE_NAME%:latest"
                    }
                }
                stage('Lint Code Source') {
                    steps {
                        echo 'Testing dashboard HTML structure using Windows findstr...'
                        bat 'findstr /C:"Smart Campus Energy Monitoring System" index.html'
                    }
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    bat "echo %PASS% | docker login -u %USER% --password-stdin"
                    bat "docker push %DOCKER_HUB_USER%/%IMAGE_NAME%:%IMAGE_TAG%"
                    bat "docker push %DOCKER_HUB_USER%/%IMAGE_NAME%:latest"
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                bat "powershell -Command \"(Get-Content deployment.yaml) -replace ':%DOCKER_HUB_USER%/%IMAGE_NAME%:latest', ':%DOCKER_HUB_USER%/%IMAGE_NAME%:%IMAGE_TAG%' | Set-Content deployment.yaml\""
                bat "kubectl apply -f deployment.yaml"
                bat "kubectl rollout status deployment/campus-energy-monitor-system"
            }
        }
    }
}
