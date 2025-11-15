Content is user-generated and unverified.
pipeline {
    agent any
    
    tools {
        nodejs 'NodeJS 7.8.0' // Має відповідати імені в Global Tool Configuration
    }
    
    environment {
        DOCKER_IMAGE_NAME = "${env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}"
        DOCKER_IMAGE_TAG = 'v1.0'
        APP_PORT = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
        CONTAINER_NAME = "${env.BRANCH_NAME == 'main' ? 'app-main' : 'app-dev'}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo "Checking out ${env.BRANCH_NAME} branch..."
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                echo "Building Node.js application..."
                sh 'npm install'
            }
        }
        
        stage('Test') {
            steps {
                echo "Running tests..."
                sh 'npm test'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                    sh """
                        docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} .
                    """
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    echo "Deploying application on port ${APP_PORT}..."
                    
                    // Зупинка та видалення старого контейнера якщо він існує
                    sh """
                        # Перевірка чи існує контейнер
                        if [ \$(docker ps -aq -f name=${CONTAINER_NAME}) ]; then
                            echo "Stopping existing container..."
                            docker stop ${CONTAINER_NAME} || true
                            echo "Removing existing container..."
                            docker rm ${CONTAINER_NAME} || true
                        fi
                    """
                    
                    // Запуск нового контейнера
                    if (env.BRANCH_NAME == 'main') {
                        sh """
                            docker run -d --name ${CONTAINER_NAME} \
                                --expose 3000 \
                                -p 3000:3000 \
                                ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                        """
                        echo "Application deployed at http://localhost:3000"
                    } else if (env.BRANCH_NAME == 'dev') {
                        sh """
                            docker run -d --name ${CONTAINER_NAME} \
                                --expose 3001 \
                                -p 3001:3000 \
                                ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                        """
                        echo "Application deployed at http://localhost:3001"
                    }
                    
                    // Перевірка статусу контейнера
                    sh "docker ps -f name=${CONTAINER_NAME}"
                }
            }
        }
    }
    
    post {
        success {
            echo "Pipeline completed successfully for ${env.BRANCH_NAME} branch!"
            echo "Application is running at http://localhost:${APP_PORT}"
        }
        failure {
            echo "Pipeline failed for ${env.BRANCH_NAME} branch!"
        }
        always {
            echo "Cleaning up workspace..."
            cleanWs()
        }
    }
}
