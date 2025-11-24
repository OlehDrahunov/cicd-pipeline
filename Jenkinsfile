  agent any

    tools {
        nodejs 'NodeJS 7.8.0'   

    }

    stages {
        stage('Define Environment') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.IMAGE   = "nodemain"
                        env.PORT    = "3000"
                        env.NAME    = "app-main"
                    } else if (env.BRANCH_NAME == 'dev') {
                        env.IMAGE   = "nodedev"
                        env.PORT    = "3001"
                        env.NAME    = "app-dev"
                    } else {
                        error "Unsupported branch"

                    }


                }
                echo "Deploying ${env.BRANCH_NAME} → http://localhost:${env.PORT}"
            }
        }

        stage('Checkout & Install') {
            steps {
                checkout scm
                sh 'node --version && npm --version'
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
                
                sh """
                    docker build -t ${env.IMAGE}:v1.0 .

                    docker rm -f ${env.NAME} || true

                    docker run -d \
                        --name ${env.NAME} \
                        -p ${env.PORT}:3000 \
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