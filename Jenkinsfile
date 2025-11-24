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
    }
    
    stages {
        stage('Deploy Application') {
            steps {
                script {
                    echo "Deploying ${DOCKER_IMAGE_NAME}:${params.IMAGE_TAG}"
                    
                    sh """
                        # Зупинка та видалення старого контейнера
                        if [ \$(docker ps -aq -f name=${CONTAINER_NAME}) ]; then
                            docker stop ${CONTAINER_NAME} || true
                            docker rm ${CONTAINER_NAME} || true
                        fi
                        
                        # Запуск нового контейнера
                        docker run -d --name ${CONTAINER_NAME} \\
                            --expose ${APP_PORT} \\
                            -p ${APP_PORT}:3000 \\
                            ${DOCKER_IMAGE_NAME}:${params.IMAGE_TAG}
                        
                        sleep 3
                    """
                    
                    echo "App deployed at http://localhost:${APP_PORT}"
                }
            }
        }
    }
    
    post {
        success {
            echo "Deployment - success"
            echo "Environment: ${params.ENVIRONMENT}"
            echo "http://localhost:${APP_PORT}"
        }
        failure {
            echo "Deployment failed for ${params.ENVIRONMENT} environment!"
        }
    }
} 
