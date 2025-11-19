pipeline {
    agent any
    
    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['main', 'dev'],
            description: 'Select environment to deploy (main or dev)'
        )
        string(
            name: 'IMAGE_TAG',
            defaultValue: 'v1.0',
            description: 'Docker image tag to deploy'
        )
    }
    
    environment {
        DOCKER_IMAGE_NAME = "${params.ENVIRONMENT == 'main' ? 'nodemain' : 'nodedev'}"
        APP_PORT = "${params.ENVIRONMENT == 'main' ? '3000' : '3001'}"
        CONTAINER_NAME = "${params.ENVIRONMENT == 'main' ? 'app-main' : 'app-dev'}"
        GIT_BRANCH = "${params.ENVIRONMENT == 'main' ? 'main' : 'dev'}"
    }
    
    stages {
        stage('Validate Parameters') {
            steps {
                script {
                    echo "Deployment Configuration"
                    echo "Environment: ${params.ENVIRONMENT}"
                    echo "Image Tag: ${params.IMAGE_TAG}"
                    echo "Docker Image: ${DOCKER_IMAGE_NAME}:${params.IMAGE_TAG}"
                    echo "Port: ${APP_PORT}"
                    echo "Container Name: ${CONTAINER_NAME}"
                    
                    // Перевірка чи існує образ
                    def imageExists = sh(
                        script: "docker images -q ${DOCKER_IMAGE_NAME}:${params.IMAGE_TAG}",
                        returnStdout: true
                    ).trim()
                    
                    if (!imageExists) {
                        error("Docker image ${DOCKER_IMAGE_NAME}:${params.IMAGE_TAG} does not exist! Please build it first.")
                    }
                }
            }
        }
        
        stage('Stop Previous Deployment') {
            steps {
                script {
                    echo "Stopping previous deployment if exists..."
                    sh """
                        # Зупинка та видалення старого контейнера
                        if [ \$(docker ps -aq -f name=${CONTAINER_NAME}) ]; then
                            echo "Stop container ${CONTAINER_NAME}"
                            docker stop ${CONTAINER_NAME} || true
                            echo "Remove ${CONTAINER_NAME}"
                            docker rm ${CONTAINER_NAME} || true
                            echo "Removed successfully"
                        else
                            echo "No previous deployment found"
                        fi
                    """
                }
            }
        }
        
        stage('Deploy Application') {
            steps {
                script {
                    echo "Deploying ${DOCKER_IMAGE_NAME}:${params.IMAGE_TAG} on port ${APP_PORT}..."
                    
                    if (params.ENVIRONMENT == 'main') {
                        sh """
                            docker run -d --name ${CONTAINER_NAME} \
                                --expose 3000 \
                                #-p 3001:8088 \
			        -p 3000:3000 \
                                --restart unless-stopped \
                                ${DOCKER_IMAGE_NAME}:${params.IMAGE_TAG}
                        """
                        echo "App deployed at http://localhost:3000"
                    } else if (params.ENVIRONMENT == 'dev') {
                        sh """
                            docker run -d --name ${CONTAINER_NAME} \
                                --expose 3001 \
                                #-p 3001:8088 \
                                -p 3001:3000 \
                                --restart unless-stopped \
                                ${DOCKER_IMAGE_NAME}:${params.IMAGE_TAG}
                        """
                        echo "App deployed at http://localhost:3001"
                    }
                                       
                    sleep(time: 5, unit: 'SECONDS')
                }
            }
        }
        
        stage('Health Check') {
            steps {
                script {
                    echo "Performing health check"
                    
                    // Перевірка чи контейнер запущений
                    def containerStatus = sh(
                        script: "docker ps -f name=${CONTAINER_NAME} --format '{{.Status}}'",
                        returnStdout: true
                    ).trim()
                    
                    if (containerStatus) {
                        echo "Container run ${containerStatus}"
                        
                        // логи контейнера
                        echo "Logs:"
                        sh "docker logs ${CONTAINER_NAME} --tail 20"
                    } else {
                        error("Container failed to start")
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo """
            Deployment - success
            Environment: ${params.ENVIRONMENT}
            Image: ${DOCKER_IMAGE_NAME}:${params.IMAGE_TAG}
            URL: http://localhost:${APP_PORT}
            Container: ${CONTAINER_NAME}
            """
        }
        failure {
            echo "Deployment failed for ${params.ENVIRONMENT} environment!"
            
            sh """
                if [ \$(docker ps -aq -f name=${CONTAINER_NAME}) ]; then
                    echo "Container logs:"
                    docker logs ${CONTAINER_NAME} || true
                fi
            """
        }
    }
}
