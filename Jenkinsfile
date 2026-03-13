pipeline {
    agent any

    environment {
        DOCKER_IMAGE   = 'ghcr.io/flaresolverr/flaresolverr:latest'
        CONTAINER_NAME = 'flaresolverr'
        LOG_LEVEL      = 'info'
        PORT           = '8191'
    }

    parameters {
        choice(name: 'ACTION', choices: ['Deploy', 'Update', 'Stop and Run'], description: 'Select action')
    }

    stages {

        stage('Pull Latest Image') {
            when {
                expression { ACTION == 'Update' }
            }
            steps {
                script {
                    sh "docker pull ${DOCKER_IMAGE}"
                }
            }
        }

        stage('Check Existing Container') {
            steps {
                script {
                    env.CONTAINER_EXISTS = sh(
                        script: "docker ps -aq -f name=${CONTAINER_NAME}",
                        returnStdout: true
                    ).trim()
                }
            }
        }

        stage('Perform Selected Action') {
            steps {
                script {

                    if (ACTION == 'Update' && CONTAINER_EXISTS) {
                        echo "Updating container ${CONTAINER_NAME}"

                        sh """
                        docker stop ${CONTAINER_NAME}
                        docker rm ${CONTAINER_NAME}
                        docker run -d \
                            --restart unless-stopped \
                            --name ${CONTAINER_NAME} \
                            -p ${PORT}:${PORT} \
                            -e LOG_LEVEL=${LOG_LEVEL} \
                            ${DOCKER_IMAGE}
                        """

                    } else if (ACTION == 'Stop and Run' && CONTAINER_EXISTS) {
                        echo "Restarting container ${CONTAINER_NAME} without updating image"

                        sh """
                        docker stop ${CONTAINER_NAME}
                        docker start ${CONTAINER_NAME}
                        """

                    } else {
                        echo "Deploying new container ${CONTAINER_NAME}"

                        sh """
                        docker run -d \
                            --restart unless-stopped \
                            --name ${CONTAINER_NAME} \
                            -p ${PORT}:${PORT} \
                            -e LOG_LEVEL=${LOG_LEVEL} \
                            ${DOCKER_IMAGE}
                        """
                    }
                }
            }
        }
    }
}
