pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "skparmar/capstone2-devops"
        DOCKER_TAG   = "latest"
        DOCKER_CREDS = "dockerhub-creds"
        KUBECONFIG   = "/var/lib/jenkins/.kube/config"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/skandparmar/capstone2-devops.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                  docker build -t $DOCKER_IMAGE:$DOCKER_TAG .
                '''
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: DOCKER_CREDS,
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                      echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                      docker push $DOCKER_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Release Validation (25th Only)') {
            steps {
                script {
                    def day = new Date().format("dd")
                    if (day != "25") {
                        error(" Release allowed only on 25th of the month")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                  kubectl apply -f k8s-deployment.yaml
                  kubectl apply -f k8s-service.yaml
                '''
            }
        }
    }

    post {
        success {
            echo " CI/CD Pipeline executed successfully"
        }
        failure {
            echo " CI/CD Pipeline failed"
        }
    }
}
