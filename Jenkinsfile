pipeline {
    agent any

    tools {
        nodejs 'NodeJS 25.2.1'   
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
                        error "Branch not supported: ${env.BRANCH_NAME}"
                    }
                }
                echo "Branch: ${env.BRANCH_NAME} → http://localhost:${env.APP_PORT}"
            }
        }

        stage('Checkout')   { steps { checkout scm } }
        
        stage('Install') {
            steps {
                sh 'node --version'
                sh 'npm --version'
                sh 'npm install' 
            }
        }

        stage('Test')       { steps { sh 'npm test || echo "No tests"' } }

        stage('Build Image') {
            steps {
                sh "docker build -t ${env.DOCKER_IMAGE}:v1.0 ."
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    docker rm -f ${env.CONTAINER_NAME} || true
                    docker run -d --name ${env.CONTAINER_NAME} -p ${env.APP_PORT}:3000 ${env.DOCKER_IMAGE}:v1.0
                    echo "Deployed: http://localhost:${env.APP_PORT}"
                """
            }
        }
    }
}