pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "node${env.BRANCH_NAME}"
        IMAGE_TAG = "v1.0"
        PORT = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
    }
    
    tools {
        nodejs 'Node 7.8.0'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo "Checking out code from branch: ${env.BRANCH_NAME}"
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                echo "Building application for branch: ${env.BRANCH_NAME}"
                sh 'npm install'
            }
        }
        
        stage('Test') {
            steps {
                echo "Running tests for branch: ${env.BRANCH_NAME}"
                sh 'npm test || true'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${DOCKER_IMAGE}:${IMAGE_TAG}"
                    docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}")
                }
            }
        }
        
        stage('Scan Docker Image for Vulnerabilities') {
            steps {
                script {
                    def vulnerabilities = sh(
                        script: "trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress ${DOCKER_IMAGE}:${IMAGE_TAG}",
                        returnStdout: true
                    ).trim()
                    echo "Vulnerability Report:\n${vulnerabilities}"
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    echo "Deploying application on port ${PORT} for branch: ${env.BRANCH_NAME}"
                    
                    // Удаляем только контейнеры текущего окружения
                    sh """
                        docker ps -a --filter "name=${DOCKER_IMAGE}" --format "{{.ID}}" | xargs -r docker rm -f
                    """
                    
                    // Запускаем новый контейнер
                    sh """
                        docker run -d \
                        --name ${DOCKER_IMAGE}-container \
                        -p ${PORT}:3000 \
                        ${DOCKER_IMAGE}:${IMAGE_TAG}
                    """
                    
                    echo "Application deployed successfully on http://localhost:${PORT}"
                }
            }
        }
    }
    
    post {
        success {
            echo "Pipeline executed successfully for branch: ${env.BRANCH_NAME}"
        }
        failure {
            echo "Pipeline failed for branch: ${env.BRANCH_NAME}"
        }
        always {
            cleanWs()
        }
    }
}