pipeline {
    agent any

    environment {
        WORK_DIR        = "${env.WORKSPACE}"
        IMAGE_NAME      = "online-bs"
        IMAGE_TAG       = "latest"
        CONTAINER_NAME  = "online-container"
        HOST_PORT       = "3000"
        DOCKERHUB_REPO  = "manikanta1319"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/Manikanta18vk/onlinebookstore.git'
            }
        }

        stage('Build Package') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Try removing existing image (no fail if missing)
                    sh 'docker image inspect ${IMAGE_NAME}:${IMAGE_TAG} >/dev/null 2>&1 || true'
                    sh 'docker rmi -f ${IMAGE_NAME}:${IMAGE_TAG} || true'
                }

                // Build Docker image from the workspace
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -f ${WORK_DIR}/Dockerfile ${WORK_DIR}'
            }
        }

        stage('Run Docker Container (Local Test)') {
            steps {
                sh '''
                    docker rm -f ${CONTAINER_NAME} || true
                    docker run -d --name ${CONTAINER_NAME} -p ${HOST_PORT}:80 ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKERHUB_REPO}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${DOCKERHUB_REPO}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker logout
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "🧹 Cleaning up Docker resources..."
            sh 'docker rm -f ${CONTAINER_NAME} || true'
            sh 'docker rmi -f ${IMAGE_NAME}:${IMAGE_TAG} || true'
        }
    }
}
