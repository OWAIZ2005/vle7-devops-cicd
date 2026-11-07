
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'owaiz2005/vle7-devops-app'
        DOCKER_CREDENTIALS = 'dockerhub-creds'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/OWAIZ2005/vle7-devops-cicd.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: DOCKER_CREDENTIALS,
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                        -u "$DOCKER_USER" --password-stdin

                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                        docker push ${DOCKER_IMAGE}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Check WSL Access') {
            steps {
                bat '''
                    echo === Jenkins Windows Account ===
                    whoami

                    echo === Checking WSL ===
                    wsl.exe -l -q

                    echo === Checking Kubernetes ===
                    wsl.exe -d Ubuntu -u gmbas -- bash -lc "kubectl get nodes"

                    echo === Checking Docker ===
                    wsl.exe -d Ubuntu -u gmbas -- bash -lc "docker --version"
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl set image deployment/vle7-demo \
                    vle7-app=${DOCKER_IMAGE}:${BUILD_NUMBER}

                    kubectl rollout status deployment/vle7-demo
                '''
            }
        }
    }
}
