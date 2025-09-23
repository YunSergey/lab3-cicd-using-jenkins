pipeline {
    agent any
    
    environment {
        // Определяем порт в зависимости от ветки
        APP_PORT = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
        // Имя Docker образа в зависимости от ветки  
        DOCKER_IMAGE = "${env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}"
        DOCKER_TAG = "v1.0"
    }
    
    tools {
        nodejs 'node' // Должно совпадать с именем в Global Tool Configuration
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo "Checking out ${env.BRANCH_NAME} branch"
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                echo "Building Node.js application for ${env.BRANCH_NAME} branch"
                sh 'npm install'
            }
        }
        
        stage('Test') {
            steps {
                echo "Running tests for ${env.BRANCH_NAME} branch"
                sh 'npm test'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    echo "Deploying to ${env.BRANCH_NAME} environment on port ${APP_PORT}"
                    
                    // Останавливаем и удаляем существующий контейнер для текущей ветки
                    sh """
                        docker stop ${DOCKER_IMAGE}-container || true
                        docker rm ${DOCKER_IMAGE}-container || true
                    """
                    
                    // Запускаем новый контейнер
                    if (env.BRANCH_NAME == 'main') {
                        sh "docker run -d --name ${DOCKER_IMAGE}-container --expose 3000 -p 3000:3000 ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    } else {
                        sh "docker run -d --name ${DOCKER_IMAGE}-container --expose 3001 -p 3001:3000 ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    }
                    
                    echo "Application deployed successfully on port ${APP_PORT}"
                    echo "Access URL: http://localhost:${APP_PORT}"
                }
            }
        }
    }
    
    post {
        always {
            echo "Pipeline execution completed for ${env.BRANCH_NAME}"
            // Очистка workspace при необходимости
            // cleanWs()
        }
        success {
            echo "Pipeline succeeded for ${env.BRANCH_NAME}!"
        }
        failure {
            echo "Pipeline failed for ${env.BRANCH_NAME}!"
        }
    }
}
