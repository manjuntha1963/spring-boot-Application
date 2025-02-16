pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'manjuntha1963/my-java-app' // Docker Hub Image
        BRANCH = 'master'                          // GitHub Branch
        KUBECONFIG = '/home/ubuntu/.kube/config'   // Path to kubeconfig
        DEPLOYMENT_NAME = 'my-java-app'            // Kubernetes Deployment Name
        GITHUB_REPO = "https://github.com/manjuntha1963/spring-boot-Application.git"
    }

    stages {
        stage('Checkout') {
            steps {
                // Clone GitHub repository
                git branch: BRANCH, url: 'https://github.com/manjuntha1963/spring-boot-Application.git'
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

        stage('Deploy to EKS') {
            steps {
                script {
                    echo 'Deploying to Kubernetes EKS...'
                    sh """
                    # Set Kubernetes context
                    export KUBECONFIG=$KUBECONFIG

                    # Apply Kubernetes manifests from GitHub repo
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    """

            
                }
            }
        }
    }
}
    }
}
