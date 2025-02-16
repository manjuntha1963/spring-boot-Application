pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'manjuntha1963/my-java-app'  // Docker Hub Image
        BRANCH = 'master'                           // GitHub Branch
        KUBECONFIG = '/var/lib/jenkins/.kube/config' // Path to kubeconfig
        DEPLOYMENT_NAME = 'my-java-app'             // Kubernetes Deployment Name
        GITHUB_REPO = "https://github.com/manjuntha1963/spring-boot-Application.git"
        SONARQUBE_SERVER = 'sonarqube'              // Jenkins SonarQube Server Name
        RECIPIENTS = 'manjuntha1963@gmail.com'       // Email recipient
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

        stage('SonarQube Analysis') {
            steps {
                script {
                    echo 'Running SonarQube analysis...'
                    withSonarQubeEnv(SONARQUBE_SERVER) {
                        sh 'mvn sonar:sonar -Dsonar.projectKey=my-java-app -Dsonar.host.url=http://54.165.163.214:9000'
                    }
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

        stage('Trivy Security Scan') {
            steps {
                script {
                    echo 'Running Trivy security scan on Docker image...'
                    sh '''
                    # Install Trivy if not installed
                    if ! command -v trivy &> /dev/null; then
                        echo "Installing Trivy..."
                        curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh
                    fi

                    # Run Trivy scan
                    trivy image --exit-code 1 --severity HIGH,CRITICAL $DOCKER_IMAGE
                    '''
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
            emailext (
                subject: "Jenkins Pipeline Success: $JOB_NAME #$BUILD_NUMBER",
                body: "The pipeline for $JOB_NAME completed successfully.\n\nCheck the build logs: ${BUILD_URL}",
                recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                to: RECIPIENTS
            )
        }
        failure {
            echo '❌ Build or Deployment failed!'
            emailext (
                subject: "Jenkins Pipeline Failed: $JOB_NAME #$BUILD_NUMBER",
                body: "The pipeline for $JOB_NAME has failed.\n\nCheck the build logs: ${BUILD_URL}",
                recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                to: RECIPIENTS
            )
        }
    }
}
