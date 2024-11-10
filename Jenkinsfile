pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'aman7ph/landing'
        VERSION = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/aman7ph/landing'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_HUB_REPO}:${VERSION} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_HUB_USERNAME', passwordVariable: 'DOCKER_HUB_PASSWORD')]) {
                        sh "docker login -u ${DOCKER_HUB_USERNAME} -p ${DOCKER_HUB_PASSWORD}"
                        sh "docker push ${DOCKER_HUB_REPO}:${VERSION}"
                    }
                }
            }
        }

        stage('Deploy on VM') {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'vm-ssh-credentials', keyFileVariable: 'SSH_KEY')]) {
                        sh """
                            ssh -i ${SSH_KEY} orc@4.233.73.60 << EOF
                            docker pull ${DOCKER_HUB_REPO}:${VERSION}
                            docker stop landing_container || true
                            docker rm landing_container || true
                            docker run -d -p 3000:3000 --name landing_container ${DOCKER_HUB_REPO}:${VERSION}
                            EOF
                        """
                    }
                }
            }
        }
    }
}
