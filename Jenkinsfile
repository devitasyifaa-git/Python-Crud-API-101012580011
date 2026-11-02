
pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        PYTHONUNBUFFERED = '1'
        REGISTRY_NAME = 'python-crud-api'
        CONTAINER_NAME = 'production-api-container'
        PORT_MAPPING = '5000:5000'
        VOLUME_NAME = 'production-api-data'
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                echo 'Checking out source code from GitHub'
                checkout scm

                script {
                    env.GIT_COMMIT_SHORT = sh(
                        script: 'git rev-parse --short=7 HEAD',
                        returnStdout: true
                    ).trim()

                    echo "Commit SHA: ${env.GIT_COMMIT_SHORT}"
                }
            }
        }

        stage('2. Code Syntax Verification') {
            steps {
                sh 'python3 -m py_compile app.py'
            }
        }

        stage('3. Build Docker Image') {
            steps {
                sh 'docker build -t ${REGISTRY_NAME}:${BUILD_NUMBER} .'

                sh 'docker tag ${REGISTRY_NAME}:${BUILD_NUMBER} ${REGISTRY_NAME}:latest'

                sh 'docker tag ${REGISTRY_NAME}:${BUILD_NUMBER} ${REGISTRY_NAME}:${GIT_COMMIT_SHORT}'

                echo "Docker Image Build Number: ${env.BUILD_NUMBER}"
                echo "Docker Image Commit SHA: ${env.GIT_COMMIT_SHORT}"

                sh 'docker images ${REGISTRY_NAME}'
            }
        }

        stage('4. Automated Deployment') {
            steps {
                sh '''
                    docker stop ${CONTAINER_NAME} || true

                    docker rm ${CONTAINER_NAME} || true

                    docker volume create ${VOLUME_NAME}

                    docker run -d \
                      --name ${CONTAINER_NAME} \
                      -p ${PORT_MAPPING} \
                      -v ${VOLUME_NAME}:/var/lib/backend_data \
                      ${REGISTRY_NAME}:latest
                '''
            }
        }

        stage('5. Health Check') {
            steps {
                sh '''
                    echo "Waiting for API Health Endpoint..."

                    for attempt in $(seq 1 15); do
                        if curl -fsS http://host.docker.internal:5000/health; then
                            echo "API Health Endpoint is responding."
                            break
                        fi

                        if [ "$attempt" -eq 15 ]; then
                            echo "API Health Check failed."
                            docker logs --tail 50 ${CONTAINER_NAME}
                            exit 1
                        fi

                        sleep 2
                    done

                    echo "Checking Docker Container Health..."

                    for attempt in $(seq 1 15); do
                        HEALTH_STATUS=$(docker inspect --format='{{.State.Health.Status}}' ${CONTAINER_NAME})

                        echo "Docker Health Status: ${HEALTH_STATUS}"

                        if [ "$HEALTH_STATUS" = "healthy" ]; then
                            echo "Docker HEALTHCHECK passed."
                            docker logs --tail 20 ${CONTAINER_NAME}
                            exit 0
                        fi

                        if [ "$HEALTH_STATUS" = "unhealthy" ]; then
                            echo "Docker HEALTHCHECK failed."
                            docker logs --tail 50 ${CONTAINER_NAME}
                            exit 1
                        fi

                        sleep 2
                    done

                    echo "Container did not become healthy in time."
                    docker logs --tail 50 ${CONTAINER_NAME}
                    exit 1
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Inspect Console Output.'
        }
    }
}
