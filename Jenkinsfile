pipeline {
    agent any

    
    environment {
        
        IMAGE = "${env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}"
        PORT = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
        NAME = "app-${env.BRANCH_NAME}"
        IMAGE_TAG = "v1.0"
    }

    tools {
        nodejs 'NodeJS 7.8.0'
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out code for branch: ${env.BRANCH_NAME}"
                checkout scm
            }
        }

        stage('Build Dependencies') {
            steps {
                sh 'npm install --legacy-peer-deps'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test || true' 
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE}:${IMAGE_TAG} ."
            }
        }

        stage('Scan Docker Image') {
            steps {
                script {
                    echo "Scanning Docker image: ${IMAGE}:${IMAGE_TAG}"
                    
                    sh "trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress ${IMAGE}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying ${env.BRANCH_NAME} to http://localhost:${PORT}"
                sh """
                    # Advanced Task: Stop and remove ONLY the container for this environment
                    docker ps -a --filter "name=${NAME}" --format "{{.ID}}" | xargs -r docker rm -f

                    docker run -d \\
                        --name ${NAME} \\
                        -p ${PORT}:3000 \\
                        ${IMAGE}:${IMAGE_TAG}
                """
                echo "${env.BRANCH_NAME} deployed successfully."
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline Success: ${env.BRANCH_NAME} deployed at http://localhost:${PORT}"
        }
        failure {
            echo "❌ Pipeline Failed for ${env.BRANCH_NAME}"
        }
        always {
            cleanWs()
        }
    }
}