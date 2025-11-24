pipeline {
    agent any
    
    tools {
        nodejs 'NodeJS 7.8.0'
    }
    
    environment {
        IMAGE_NAME = "${env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}"
        IMAGE_TAG = "v1.0"
        PORT = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
        CONTAINER = "${env.BRANCH_NAME == 'main' ? 'app-main' : 'app-dev'}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo "Checkout ${env.BRANCH_NAME} branch"
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
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }
        
        stage('Deploy') {
            steps {
                sh """
                    docker stop ${CONTAINER} || true
                    docker rm ${CONTAINER} || true
                    docker run -d --name ${CONTAINER} -p ${PORT}:3000 ${IMAGE_NAME}:${IMAGE_TAG}
                """
                echo "App at http://localhost:${PORT}"
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}
