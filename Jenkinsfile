pipeline {
    agent {
        label 'sonarqube'
    }

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
        REPO_URL = "https://github.com/Balachandru-ai/fullstack-project.git"
        GIT_BRANCH = "main"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                sh '''
                docker build -t employee-backend:${IMAGE_TAG} ./backend
                '''
            }
        }

        stage('Build Frontend Docker Image') {
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
                withCredentials([
                    usernamePassword(
                        credentialsId: 'github-creds',
                        usernameVariable: 'USERNAME',
                        passwordVariable: 'TOKEN'
                    )
                ]) {

                    sh '''
                    git config user.name "Jenkins"
                    git config user.email "jenkins@example.com"

                    git remote set-url origin https://${USERNAME}:${TOKEN}@github.com/Balachandru-ai/fullstack-project.git

                    git fetch origin

                    git checkout -B main origin/main

                    git add k8s/

                    if git diff --cached --quiet
                    then
                        echo "No changes to commit."
                    else
                        git commit -m "Deploy Build ${IMAGE_TAG}"

                        git pull --rebase origin main

                        git push origin main
                    fi
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                kubectl get deployments
                kubectl get pods
                kubectl get svc
                '''
            }
        }
    }

    post {

        success {
            echo "======================================"
            echo "Deployment Successful"
            echo "ArgoCD will sync automatically"
            echo "======================================"
        }

        failure {
            echo "======================================"
            echo "Deployment Failed"
            echo "======================================"
        }
    }
}
