pipeline {
    agent any

    environment {
        APP_NAME = 'new-app-nti'
        REPO_URL = "https://github.com/ahmedishanab2-hue/k8s-CI.git"
        KUBECONFIG = '/var/jenkins_home/.kube/config'
        PATH = "/var/jenkins_home:${env.PATH}"
    }

    stages {
        stage('Getting Repo files') {
            steps {
                git branch: "main-k8s", credentialsId: 'jenkins', url: "${REPO_URL}"
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
                    withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {

                        sh """
                            echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin

                            docker tag ${APP_NAME}:${BUILD_NUMBER} ${DOCKER_USER}/${APP_NAME}:${BUILD_NUMBER}

                            docker push ${DOCKER_USER}/${APP_NAME}:${BUILD_NUMBER}
                        """

                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            ls -l
                            sed 's|IMAGE_PLACEHOLDER|${DOCKER_USER}/${APP_NAME}:${BUILD_NUMBER}|' k8s/deployment.yaml > deployment.yaml

                            kubectl apply -f deployment.yaml
                            kubectl apply -f k8s/service.yaml
                        """
                    }
                }
            }
        }
    }
}
