pipeline {
    agent {
        label 'sonarqube'
    }

    stages {

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'

                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=fullstack-project \
                            -Dsonar.projectName=fullstack-project \
                            -Dsonar.sources=backend,frontend/src \
                            -Dsonar.sourceEncoding=UTF-8 \
                            -Dsonar.exclusions=backend/venv/**,backend/**/__pycache__/**,backend/**/*.pyc,frontend/node_modules/**,frontend/dist/**
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Backend') {
            steps {
                sh '''
                    set -e

                    cd backend

                    echo "Creating Python virtual environment..."

                    rm -rf venv
                    python3.11 -m venv venv

                    echo "Installing Python dependencies..."

                    ./venv/bin/python -m pip install --upgrade pip
                    ./venv/bin/pip install -r requirements.txt

                    echo "Backend build completed"
                '''
            }
        }

        stage('Build Frontend') {
            steps {
                sh '''
                    set -e

                    cd frontend

                    echo "Installing frontend dependencies..."

                    npm install

                    echo "Building frontend..."

                    npm run build

                    echo "Frontend build completed"
                '''
            }
        }

        stage('Deploy Frontend') {
            steps {
                sh '''
                    set -e

                    echo "Deploying frontend..."

                    sudo rm -rf /usr/share/nginx/html/*
                    sudo cp -r frontend/dist/* /usr/share/nginx/html/

                    echo "Frontend deployment completed"
                '''
            }
        }

        stage('Restart Backend') {
            steps {
                sh '''
                    set -e

                    echo "Restarting FastAPI..."

                    sudo systemctl restart fastapi
                    sudo systemctl status fastapi --no-pager

                    echo "FastAPI restarted successfully"
                '''
            }
        }

        stage('Restart Nginx') {
            steps {
                sh '''
                    set -e

                    echo "Restarting Nginx..."

                    sudo systemctl restart nginx
                    sudo systemctl status nginx --no-pager

                    echo "Nginx restarted successfully"
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    set -e

                    echo "================================="
                    echo "Checking Nginx..."
                    echo "================================="

                    curl -f http://localhost

                    echo ""
                    echo "================================="
                    echo "Checking FastAPI..."
                    echo "================================="

                    curl -f http://localhost:8000/

                    echo ""
                    echo "================================="
                    echo "Health check completed successfully"
                    echo "================================="
                '''
            }
        }
    }

    post {
        success {
            echo '================================='
            echo 'Deployment Successful'
            echo '================================='
        }

        failure {
            echo '================================='
            echo 'Deployment Failed'
            echo '================================='
        }
    }
}
