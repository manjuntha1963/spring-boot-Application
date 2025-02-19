pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "manjuntha1963/my-java-app:${BUILD_NUMBER}"
        LATEST_IMAGE = "manjuntha1963/my-java-app:latest"
        BRANCH = 'master'                           // GitHub Branch
        KUBECONFIG = '/var/lib/jenkins/.kube/config' // Path to kubeconfig
        DEPLOYMENT_NAME = 'my-java-app'             // Kubernetes Deployment Name
        GITHUB_REPO = "https://github.com/manjuntha1963/spring-boot-Application.git"
        SONARQUBE_SERVER = 'sonarqube'              // Jenkins SonarQube Server Name
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Cloning GitHub repository...'
                git branch: "${BRANCH}", url: "${GITHUB_REPO}"
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

        stage('SonarQube Analysis') {
            steps {
                script {
                    echo 'Running SonarQube analysis...'
                    withSonarQubeEnv(SONARQUBE_SERVER) {
                        sh 'mvn sonar:sonar -Dsonar.projectKey=my-java-app -Dsonar.host.url=http://http://54.210.84.183:9000'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image...'
                    sh "docker build -t $DOCKER_IMAGE -t $LATEST_IMAGE ."
                }
            }
        }

        stage('Trivy Security Scan') {
            steps {
                script {
                    echo 'Running Trivy scan'
                    sh "trivy image --exit-code 1 --severity HIGH,CRITICAL \"$DOCKER_IMAGE\" || true"
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    echo 'Pushing Docker image to Docker Hub...'
                    docker.withRegistry('', 'docker-hub-credentials') {
                        sh "docker push $DOCKER_IMAGE"
                        sh "docker push $LATEST_IMAGE"
                    }
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    echo 'Deploying to Kubernetes EKS...'
                    sh """
                    export KUBECONFIG=$KUBECONFIG
                    kubectl cluster-info
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    kubectl get pods -o wide
                    kubectl get svc my-java-app-service -o wide
                    """
                }
            }
        }
        stage('Force Kubernetes Redeployment') {
            steps {
                script {
                    echo 'Forcing Kubernetes to redeploy...'
                    sh """
                    export KUBECONFIG=$KUBECONFIG
                    kubectl set image deployment/${DEPLOYMENT_NAME} my-java-app=${LATEST_IMAGE} --record
                    """
                }
            }
        }
        stage('Deploy Prometheus and Grafana with LoadBalancer') {
            steps {
                script {
                    echo 'Deploying Prometheus and Grafana monitoring to EKS with LoadBalancer...'
                    sh """
                    export KUBECONFIG=$KUBECONFIG
                    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
                    helm repo update
                    helm upgrade --install prometheus prometheus-community/kube-prometheus-stack \
                        --namespace monitoring --create-namespace \
                        --set grafana.service.type=LoadBalancer \
                        --set prometheus.service.type=LoadBalancer --wait
                    kubectl get pods -n monitoring
                    kubectl get svc -n monitoring | grep grafana
                    kubectl get svc -n monitoring | grep prometheus
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment to EKS and monitoring setup successful!'
        }
        failure {
            echo '❌ Build or Deployment failed!'
        }
        cleanup {
            echo '🧹 Cleaning up workspace and Docker images...'
            sh "docker image prune -f"
        }
    }
}
