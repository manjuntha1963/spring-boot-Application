pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'manjuntha1963/my-java-app' // Full Docker image name
        EC2_HOST = '54.234.97.114'               // Replace with your EC2 instance's public IP
        EC2_USER = 'ubuntu'                     // EC2 username (e.g., 'ubuntu' for Ubuntu)
        BRANCH = 'master'                       // Configurable branch
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from GitHub
                git branch: BRANCH, url: 'https://github.com/manjuntha1963/springboot.git'
            }
        }

        stage('Build with Maven') {
            steps {
                script {
                    echo 'Running Maven build...'
                    sh 'mvn clean package'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image...'
                    sh "docker build -t $DOCKER_IMAGE ."
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    echo 'Pushing Docker image to Docker Hub...'
                    docker.withRegistry('', 'docker-hub-credentials') {
                        sh "docker push $DOCKER_IMAGE"
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    echo 'Deploying to EC2 instance...'
                }
                sshagent(['ssh-key-id']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no $EC2_USER@$EC2_HOST <<EOF
docker pull $DOCKER_IMAGE:latest
docker stop my-java-app || true
docker rm my-java-app || true
docker run -dp 8090:8080 --name my-java-app $DOCKER_IMAGE:latest
EOF
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Build or Deployment failed!'
        }
    }
}
