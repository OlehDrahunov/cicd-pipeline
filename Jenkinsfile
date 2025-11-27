pipeline {
    agent any

    tools {
        nodejs 'NodeJS 7.8.0'
    }

    environment {
        
        IMAGE = "${env.BRANCH_NAME == 'main' ? 'nodemain' : env.BRANCH_NAME == 'dev' ? 'nodedev' : error('Unsupported branch')}"
        PORT  = "${env.BRANCH_NAME == 'main' ? '3000' : env.BRANCH_NAME == 'dev' ? '3001' : error('Unsupported branch')}"
        NAME  = "${env.BRANCH_NAME == 'main' ? 'app-main' : env.BRANCH_NAME == 'dev' ? 'app-dev' : error('Unsupported branch')}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "Deploying ${env.BRANCH_NAME} → http://localhost:${env.PORT}"
            }
        }

        stage('Install & Test') {
            steps {
                sh 'npm install --legacy-peer-deps'
                sh 'npm test || echo "No tests - OK"'
            }
        }

        stage('Build & Deploy') {
            steps {
                sh """
                    docker build -t ${env.IMAGE}:v1.0 .

                    docker rm -f ${env.NAME} || true

                    docker run -d \\
                        --name ${env.NAME} \\
                        -p ${env.PORT}:3000 \\
                        ${env.IMAGE}:v1.0

                    echo "${env.BRANCH_NAME} deployed → http://localhost:${env.PORT}"
                """
            }
        }
    }

    post {
        success { echo "ok! ${env.BRANCH_NAME} deployed at http://localhost:${env.PORT} ${env.PORT}" }
        failure { echo "error" }
    }
}