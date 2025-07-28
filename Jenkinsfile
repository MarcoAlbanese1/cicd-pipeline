pipeline {
    agent any
    
    tools {
        node 'Node 7.8.0'
    }
    
    environment {
        DOCKER_IMAGE = "${env.BRANCH_NAME == 'main' ? 'nodemain:v1.0' : 'nodedev:v1.0'}"
        APP_PORT = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
        CONTAINER_NAME = "${env.BRANCH_NAME == 'main' ? 'node-main' : 'node-dev'}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo "Checking out branch: ${env.BRANCH_NAME}"
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                echo 'Building the application...'
                sh 'npm install'
            }
        }
        
        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'npm test'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: ${DOCKER_IMAGE}"
                script {
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }
        
        stage('Deploy') {
            steps {
                echo "Deploying to ${env.BRANCH_NAME} environment on port ${APP_PORT}"
                script {
                    sh """
                        docker stop ${CONTAINER_NAME} || true
                        docker rm ${CONTAINER_NAME} || true
                    """
                    
                    if (env.BRANCH_NAME == 'main') {
                        sh "docker run -d --name ${CONTAINER_NAME} --expose 3000 -p 3000:3000 ${DOCKER_IMAGE}"
                    } else if (env.BRANCH_NAME == 'dev') {
                        sh "docker run -d --name ${CONTAINER_NAME} --expose 3001 -p 3001:3000 ${DOCKER_IMAGE}"
                    }

                    sleep(time: 15, unit: 'SECONDS')
                    
                    sh "docker ps | grep ${CONTAINER_NAME}"
                    echo "Application deployed successfully at http://localhost:${APP_PORT}"
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            cleanWs()
        }
    }
}
