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
	stage('Build package') {
		steps {
			sh 'mvn clean package'
	}
	}

        stage('Build Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                
                        docker rmi -f ${IMAGE_NAME}:${IMAGE_TAG} || true
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -f ${WORK_DIR}/Dockerfile ${WORK_DIR}
                        docker logout
                    '''
                }
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
}
