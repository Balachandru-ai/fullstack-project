pipeline {
    agent {
        label 'fastapi'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Premchand-96/fullstack-project.git'
            }
        }

        stage('Build Backend') {
            steps {
                sh '''
                cd backend

                rm -rf venv
                python3.11 -m venv venv

                ./venv/bin/python -m pip install --upgrade pip
                ./venv/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Build Frontend') {
            steps {
                sh '''
                cd frontend
                npm install
                npm run build
                '''
            }
        }

        stage('Deploy Frontend') {
            steps {
                sh '''
                sudo rm -rf /usr/share/nginx/html/*
                sudo cp -r frontend/dist/* /usr/share/nginx/html/
                '''
            }
        }

        stage('Restart Backend') {
            steps {
                sh '''
                sudo systemctl restart fastapi
                sudo systemctl status fastapi --no-pager
                '''
            }
        }

        stage('Restart Nginx') {
            steps {
                sh '''
                sudo systemctl restart nginx
                sudo systemctl status nginx --no-pager
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                curl -I http://localhost
                curl http://localhost:8000/
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful'
        }

        failure {
            echo 'Deployment Failed'
        }
    }
}
