pipeline {
    agent any

    tools {
        nodejs 'node-7'
    }

    environment {
        IMAGE_NAME = ''
        APP_PORT  = ''
        CONTAINER_NAME = ''
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Set environment variables') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.APP_PORT = '3000'
                        env.IMAGE_NAME = 'nodemain:v1.0'
                        env.CONTAINER_NAME = 'node-main'
                    } else if (env.BRANCH_NAME == 'dev') {
                        env.APP_PORT = '3001'
                        env.IMAGE_NAME = 'nodedev:v1.0'
                        env.CONTAINER_NAME = 'node-dev'
                    } else {
                        error "Branch no soportada: ${env.BRANCH_NAME}"
                    }

                    echo "Branch: ${env.BRANCH_NAME}"
                    echo "Puerto: ${env.APP_PORT}"
                    echo "Imagen: ${env.IMAGE_NAME}"
                }
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build || echo "No build step defined"'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test || echo "No tests defined"'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                  docker build -t ${IMAGE_NAME} .
                """
            }
        }

        stage('Deploy') {
            steps {
                sh """
                  docker rm -f ${CONTAINER_NAME} || true
                  docker run -d \\
                    --name ${CONTAINER_NAME} \\
                    -e PORT=${APP_PORT} \\
                    -p ${APP_PORT}:${APP_PORT} \\
                    ${IMAGE_NAME}
                """
            }
        }
    }

    post {
        success {
            echo "Deploy exitoso para ${env.BRANCH_NAME} en puerto ${APP_PORT}"
        }
        failure {
            echo "Pipeline falló para ${env.BRANCH_NAME}"
        }
    }
}
