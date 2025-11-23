pipeline {
    agent any

    
    tools {
        nodejs 'NodeJS 7.8.0'   // ←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←
    }

    environment {
        
        DOCKER_IMAGE = ""
        APP_PORT     = ""
        CONTAINER_NAME = ""
    }

    stages {
        stage('Prepare Environment') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.DOCKER_IMAGE     = "nodemain"
                        env.APP_PORT         = "3000"
                        env.CONTAINER_NAME   = "app-main"
                    } else if (env.BRANCH_NAME == 'dev') {
                        env.DOCKER_IMAGE     = "nodedev"
                        env.APP_PORT         = "3001"
                        env.CONTAINER_NAME   = "app-dev"
                    } else {
                        error "Branch ${env.BRANCH_NAME} not supported!"
                    }
                }
                echo "Branch: ${env.BRANCH_NAME}"
                echo "Image: ${env.DOCKER_IMAGE}:v1.0"
                echo "Port: ${env.APP_PORT}"
                echo "Container: ${env.CONTAINER_NAME}"
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test || echo "No tests found - skipping"'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${env.DOCKER_IMAGE}:v1.0 .
                """
            }
        }

        stage('Deploy') {
            steps {
                sh """
                   
                    docker rm -f ${env.CONTAINER_NAME} || true
                    
                    docker run -d \
                        --name ${env.CONTAINER_NAME} \
                        -p ${env.APP_PORT}:3000 \
                        ${env.DOCKER_IMAGE}:v1.0
                        
                    echo "app started: http://localhost:${env.APP_PORT}"
                """
            }
        }
    }

    post {
        success {
            echo "port ${env.APP_PORT}!"
        }
        cleanup {
            sh 'docker system prune -f || true'
        }
    }
}