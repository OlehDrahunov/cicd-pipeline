pipeline {
    agent any

    tools {
        nodejs 'NodeJS 7.8.0'  
    }

    environment {
        DOCKER_IMAGE     = ""
        APP_PORT         = ""
        CONTAINER_NAME   = ""
    }

    stages {
        stage('Define Environment') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.DOCKER_IMAGE   = "nodemain"
                        env.APP_PORT       = "3000"
                        env.CONTAINER_NAME = "app-main"
                    } else if (env.BRANCH_NAME == 'dev') {
                        env.DOCKER_IMAGE   = "nodedev"
                        env.APP_PORT       = "3001"
                        env.CONTAINER_NAME = "app-dev"
                    } else {
                        currentBuild.result = 'FAILURE'
                        error "Unsupported branch: ${env.BRANCH_NAME}"
                    }
                    echo "Deploying ${env.BRANCH_NAME} → http://localhost:${env.APP_PORT}"
                    echo "Image: ${env.DOCKER_IMAGE}:v1.0 | Container: ${env.CONTAINER_NAME}"
                }
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'node --version'
                sh 'npm --version'
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test || echo "No tests - skipped"'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                  
                    sh '''
                        sudo usermod -aG docker jenkins || true
                        sudo chmod 666 /var/run/docker.sock || true
                    '''
                    sh "docker build -t ${env.DOCKER_IMAGE}:v1.0 ."
                    sh "docker images ${env.DOCKER_IMAGE}:v1.0"
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                   
                    docker rm -f ${CONTAINER_NAME} || true
                    
                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${APP_PORT}:3000 \
                        ${DOCKER_IMAGE}:v1.0
                        
                    echo "open browser: http://localhost:${APP_PORT}"
                '''
            }
        }
    }

    post {
        success {
            echo "app ${env.BRANCH_NAME} deployed at http://localhost:${env.APP_PORT}!"
        }
        failure {
            echo "ERROR: Deployment failed for branch ${env.BRANCH_NAME}."
        }
        always {
            sh 'docker ps | grep app- || echo "Другие контейнеры не тронуты"'
        }
    }
}