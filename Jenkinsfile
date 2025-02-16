pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'manjuntha1963/my-java-app' // Docker Hub Image
        BRANCH = 'master'                          // GitHub Branch
        KUBECONFIG = '/home/ubuntu/.kube/config'   // Path to kubeconfig
        DEPLOYMENT_NAME = 'my-java-app'            // Kubernetes Deployment Name
        NAMESPACE = 'default'                      // Kubernetes Namespace
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

                    # Create deployment.yaml dynamically
                    cat <<EOF > deployment.yaml
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: $DEPLOYMENT_NAME
                      namespace: $NAMESPACE
                    spec:
                      replicas: 2
                      selector:
                        matchLabels:
                          app: $DEPLOYMENT_NAME
                      template:
                        metadata:
                          labels:
                            app: $DEPLOYMENT_NAME
                        spec:
                          containers:
                            - name: my-java-app
                              image: $DOCKER_IMAGE:latest
                              ports:
                                - containerPort: 8080
                    EOF

                    # Apply deployment
                    kubectl apply -f deployment.yaml

                    # Create service.yaml dynamically
                    cat <<EOF > service.yaml
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: my-java-app-service
                      namespace: $NAMESPACE
                    spec:
                      selector:
                        app: $DEPLOYMENT_NAME
                      ports:
                        - protocol: TCP
                          port: 80
                          targetPort: 8080
                      type: LoadBalancer
                    EOF

                    # Apply service
                    kubectl apply -f service.yaml

                    # Get LoadBalancer URL
                    kubectl get svc my-java-app-service -o wide
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment to EKS successful!'
        }
        failure {
            echo 'Build or Deployment failed!'
        }
    }
}
