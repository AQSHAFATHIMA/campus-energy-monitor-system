pipeline {
    agent any

    options {
        timestamps()
    }

    environment {
        DOCKER_IMAGE = '78619/campus-energy'
        K8S_DEPLOYMENT = 'smart-campus-energy'
        K8S_CONTAINER = 'energy-dashboard'
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo 'Checking out Smart Campus Energy project'
                bat 'dir'
            }
        }

        stage('Build & Test Parallel') {
            parallel {

                stage('Build Docker Image') {
                    steps {
                        echo 'Building Docker image'
                        bat 'docker build -t 78619/campus-energy:v1 .'
                    }
                }

                stage('Lint Code Source') {
                    steps {
                        echo 'Checking HTML and Dockerfile'

                        bat 'if not exist index.html exit /b 1'
                        bat 'if not exist Dockerfile exit /b 1'
                        bat 'findstr /i "<html" index.html'

                        echo 'HTML and Dockerfile checks passed'
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DH_USER',
                        passwordVariable: 'DH_PASS'
                    )
                ]) {
                    bat '''
                        @echo off
                        echo %DH_PASS%| docker login -u %DH_USER% --password-stdin
                        if errorlevel 1 exit /b 1
                    '''
                }

                bat 'docker push 78619/campus-energy:v1'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    try {
                        bat 'kubectl apply -f deployment.yaml'

                        bat 'kubectl rollout status deployment/smart-campus-energy --timeout=120s'

                        echo 'Deployment successful'

                    } catch (err) {
                        echo 'Deployment failed. Attempting rollback.'

                        bat 'kubectl rollout undo deployment/smart-campus-energy'

                        bat 'kubectl rollout status deployment/smart-campus-energy --timeout=120s'

                        error('Deployment failed. Rollback attempted.')
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                bat 'kubectl get deployments'
                bat 'kubectl get pods'
                bat 'kubectl get services'
            }
        }

        stage('Archive Build Artifacts') {
            steps {
                archiveArtifacts(
                    artifacts: 'index.html,Dockerfile,Jenkinsfile,deployment.yaml',
                    fingerprint: true
                )
            }
        }
    }

    post {
        success {
            echo 'Smart Campus Energy pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check Console Output.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}
```
