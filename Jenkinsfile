pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'dhidr/students-management'
        DOCKER_TAG = "${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                echo '====== Checking out code from GitHub ======'
                git branch: 'main', 
                    url: 'https://github.com/dd1-u/StudentsManagement-DevOps.git'
            }
        }
        
        stage('Build with Maven') {
            steps {
                echo '====== Building application with Maven ======'
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo '====== Building Docker image ======'
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                    docker.build("${DOCKER_IMAGE}:latest")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                echo '====== Pushing Docker image to DockerHub ======'
                script {
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_CREDENTIALS_ID) {
                        dockerImage.push("${DOCKER_TAG}")
                        dockerImage.push("latest")
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                echo '====== Deploying to Kubernetes cluster ======'
                sh '''
                    kubectl set image deployment/spring-deployment \
                        spring-container=${DOCKER_IMAGE}:${DOCKER_TAG} \
                        -n devops
                    
                    kubectl rollout status deployment/spring-deployment -n devops
                    
                    kubectl get pods -n devops
                '''
            }
        }
        
        stage('Verify Deployment') {
            steps {
                echo '====== Verifying deployment ======'
                sh '''
                    echo "Checking pods status..."
                    kubectl get pods -n devops | grep spring-deployment
                    
                    echo "Checking service..."
                    kubectl get svc spring-service -n devops
                    
                    echo "Deployment successful!"
                '''
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline completed successfully!'
            echo "🚀 Application deployed with image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
        }
        failure {
            echo '❌ Pipeline failed. Check logs for details.'
        }
        always {
            echo '🧹 Cleaning up workspace...'
            cleanWs()
        }
    }
}
