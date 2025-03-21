pipeline {
    agent any

    triggers {
        pollSCM '*/5 * * * *'  // Poll GitHub every 5 minutes
    }

    environment {
        DOCKER_CREDENTIALS_ID = 'proj-jenkins'  // Jenkins credentials ID for GitHub authentication
        GITHUB_REPO = 'https://github.com/ayusao/VIZUO.git'  // GitHub repository URL
        DOCKER_IMAGE = "vizuo/vizuo:latest"  // Docker image name
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    echo "Cloning the repository from GitHub..."
                    git branch: 'main', credentialsId: "${DOCKER_CREDENTIALS_ID}", url: "${GITHUB_REPO}"
                }
            }
        }

        stage('Build and Push Docker Image') {
            agent {
                docker {
                    image 'docker:latest'
                    args '--privileged -v /var/run/docker.sock:/var/run/docker.sock -e HOME=/tmp'
                    reuseNode true
                }
            }
            steps {
                script {
                    echo "Building Docker image: $DOCKER_IMAGE"
                    sh "docker build -t $DOCKER_IMAGE ."
                    
                    echo "Pushing Docker image to the registry..."
                    sh "docker push $DOCKER_IMAGE"  // Optionally, push to a registry
                }
            }
        }

        stage('Deploy using Docker Compose') {
            agent {
                docker {
                    image 'docker:latest'
                    args '--privileged -v /var/run/docker.sock:/var/run/docker.sock -e HOME=/tmp'
                    reuseNode true
                }
            }
            steps {
                script {
                    echo "Deploying using Docker Compose..."

                    // Build and start the containers using Docker Compose
                    sh 'docker-compose -f docker-compose.yml up --build -d'

                    // Optionally, you can add a command to check container status or logs
                    sh 'docker-compose ps'  // List running containers
                    // sh 'docker-compose logs'  // View logs (optional)
                }
            }
        }
    }

    post {
        success {
            echo "CI/CD Pipeline completed successfully!"
        }
        failure {
            echo "CI/CD Pipeline failed!"
        }
    }
}
