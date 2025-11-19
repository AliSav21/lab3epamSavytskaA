pipeline {
    agent any
    
    tools {
        nodejs 'NodeJS 7.8.0'
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
                echo "Check ${env.BRANCH_NAME} branch"
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                echo "Build Node.js app"
                sh 'npm install'
            }
        }
        
        stage('Test') {
            steps {
                echo "Running tests"
                sh 'npm test'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Build Docker image: ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                    sh """
                        docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} .
                    """
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    echo "Deploy app on port ${APP_PORT}"
                    
                    sh """
                        # Перевірка чи існує контейнер
                        if [ \$(docker ps -aq -f name=${CONTAINER_NAME}) ]; then
                            #echo "Stopping existing container"
                            docker stop ${CONTAINER_NAME} || true
                            #echo "Removing container"
                            docker rm ${CONTAINER_NAME} || true
                        fi
                    """
                    
                    if (env.BRANCH_NAME == 'main') {
                        sh """
                            docker run -d --name ${CONTAINER_NAME} \
                                #--expose 3000 \
                                -p {APP_PORT}:8080
				#-p 3000:3000 \
                                ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                        """
                        echo "Application at http://localhost:3000"
                    } else if (env.BRANCH_NAME == 'dev') {
                        sh """
                            docker run -d --name ${CONTAINER_NAME} \
                                --expose 3001 \
                                -p 3001:3000 \
                                ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                        """
                        echo "Application at http://localhost:3001"
                    }
                    
                    sh "docker ps -f name=${CONTAINER_NAME}"
                }
            }
        }
    }
    
    post {
        success {
            echo "Pipeline - success at ${env.BRANCH_NAME} branch!"
            echo "App run at http://localhost:${APP_PORT}"
        }
        failure {
            echo "Pipeline failed in ${env.BRANCH_NAME} branch!"
        }
        always {
            echo "Clean workspace."
            cleanWs()
        }
    }
}
