pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'manjuntha1963/my-java-app' // Docker Hub Image
        BRANCH = 'master'                          // GitHub Branch
        KUBECONFIG = '/home/jenkins/.kube/config'   // Path to kubeconfig
        DEPLOYMENT_NAME = 'my-java-app'            // Kubernetes Deployment Name
        GITHUB_REPO = "https://github.com/manjuntha1963/spring-boot-Application.git"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Cloning GitHub repository...'
                git branch: BRANCH, url: GITHUB_REPO
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

                    # Ensure kubectl is working
                    kubectl cluster-info
                    
                    # Apply Kubernetes manifests
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    
                    # Verify deployment
                    kubectl get pods -o wide
                    kubectl get svc my-java-app-service -o wide
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment to EKS successful!'
        }
        failure {
            echo '❌ Build or Deployment failed!'
        }
    }
}
