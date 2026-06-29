pipeline {
    agent any

    environment {
        APP_NAME = 'node-app'
        REPO_URL = "https://github.com/ahmeddhussain/multi-branch-jenkins.git"
        PORT = '3000'
    }

    stages {
        stage('Getting Repo files') {  
            steps {
                git branch: "${GIT_BRANCH}", credentialsId: 'github', url: "${REPO_URL}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        docker build -t ${APP_NAME}:${BUILD_NUMBER} .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh """
                            echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin
                            docker tag ${APP_NAME}:${BUILD_NUMBER} ${DOCKER_USERNAME}/${APP_NAME}:${BUILD_NUMBER}
                            docker push ${DOCKER_USERNAME}/${APP_NAME}:${BUILD_NUMBER}
                        """
                    }
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    sh """
                        # Stop and remove any previous containers with the same app name to prevent conflicts
                        docker ps -q --filter "name=${APP_NAME}-main" | xargs -r docker stop
                        docker ps -a -q --filter "name=${APP_NAME}-main" | xargs -r docker rm

                        # Run the new container
                        docker run -p ${PORT}:${PORT} --name "${APP_NAME}"-"main"-${BUILD_NUMBER} -d ${APP_NAME}:${BUILD_NUMBER}
                        docker ps
                    """
                }
            }
        }
    }
}
