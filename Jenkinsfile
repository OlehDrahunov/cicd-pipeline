pipeline {
    agent any
    tools {
        nodejs 'Node 7.8.0'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.PORT = 3000
                        env.IMAGE_NAME = "nodemain:v1.0"
                    } else if (env.BRANCH_NAME == 'dev') {
                        env.PORT = 3001
                        env.IMAGE_NAME = "nodedev:v1.0"
                    }
                    sh "docker build -t ${env.IMAGE_NAME} ."
                }
            }
        }
        
        stage('Scan Docker Image for Vulnerabilities') {
            steps {
                script {
                    def vulnerabilities = sh(script: "trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress ${env.IMAGE_NAME}", returnStdout: true).trim()
                    echo "Vulnerability Report:\n${vulnerabilities}"
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    
                    sh "docker stop ${env.IMAGE_NAME} || true"
                    sh "docker rm ${env.IMAGE_NAME} || true"
                    sh "docker run -d -p ${env.PORT}:3000 --name ${env.IMAGE_NAME} ${env.IMAGE_NAME}"
                }
            }
        }
    }
    
    post {
        always {
            echo "Pipeline execution completed for branch: ${env.BRANCH_NAME}"
        }
    }
}