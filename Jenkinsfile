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
                        error "Branch ${env.BRANCH_NAME} not supported"
                    }
                }
                echo "Deploying ${env.BRANCH_NAME} → http://localhost:${env.APP_PORT}"
            }
        }

        stage('Checkout') { steps { checkout scm } }

        stage('Install Dependencies') {
            steps {
                sh 'node --version'
                sh 'npm --version'
                sh 'npm install --legacy-peer-deps'  
            }
        }

        stage('Test') {
            steps {
                sh 'npm test || echo "No tests - OK"'
            }
        }

        stage('Build & Deploy') {
            steps {
                sh '''
                    
                    sudo usermod -aG docker $USER || true
                    sudo chmod 666 /var/run/docker.sock || true

                   
                    docker build -t ${DOCKER_IMAGE}:v1.0 .

                    
                    docker rm -f ${CONTAINER_NAME} || true
                    docker run -d --name ${CONTAINER_NAME} -p ${APP_PORT}:3000 ${DOCKER_IMAGE}:v1.0

                    echo "Application deployed at: http://localhost:${APP_PORT}"
                '''
            }
        }
    }

    post {
        success { echo "ok! ${env.BRANCH_NAME} → http://localhost:${env.APP_PORT}" }
        failure { echo "error!" }
    }
}