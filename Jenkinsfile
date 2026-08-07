pipeline {
    agent {
        label 'sonarqube'
    }

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Backend Image') {
            steps {
                sh '''
                docker build -t employee-backend:${IMAGE_TAG} ./backend
                '''
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh '''
                docker build -t employee-frontend:${IMAGE_TAG} ./frontend
                '''
            }
        }

        stage('Load Images into Minikube') {
            steps {
                sh '''
                minikube image load employee-backend:${IMAGE_TAG}
                minikube image load employee-frontend:${IMAGE_TAG}
                '''
            }
        }

        stage('Update Kubernetes YAML') {
            steps {
                sh '''
                sed -i "s|image: employee-backend:.*|image: employee-backend:${IMAGE_TAG}|g" k8s/backend-deployment.yaml

                sed -i "s|image: employee-frontend:.*|image: employee-frontend:${IMAGE_TAG}|g" k8s/frontend-deployment.yaml
                '''
            }
        }

        stage('Commit & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-creds',
                                                 usernameVariable: 'USERNAME',
                                                 passwordVariable: 'TOKEN')]) {

                    sh '''
                    git config user.name "Jenkins"
                    git config user.email "jenkins@example.com"

                    git add k8s/

                    git diff --cached --quiet || git commit -m "Deploy Build ${IMAGE_TAG}"

                    git push https://${USERNAME}:${TOKEN}@github.com/Balachandru-ai/fullstack-project.git main
                    '''
                }
            }
        }

    }

    post {

        success {
            echo "Deployment Successful"
        }

        failure {
            echo "Deployment Failed"
        }

    }
}
