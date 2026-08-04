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
                            -Dsonar.sourceEncoding=UTF-8
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
                    set -e

                    cd frontend

                    npm install
                    npm run build
                '''
            }
        }

        stage('Deploy Frontend') {
            steps {
                sh '''
                    set -e

                    sudo rm -rf /usr/share/nginx/html/*
                    sudo cp -r frontend/dist/* /usr/share/nginx/html/
                '''
            }
        }

        stage('Restart Backend') {
            steps {
                sh '''
                    set -e

                    sudo systemctl restart fastapi
                    sudo systemctl status fastapi --no-pager
                '''
            }
        }

        stage('Restart Nginx') {
            steps {
                sh '''
                    set -e

                    sudo systemctl restart nginx
                    sudo systemctl status nginx --no-pager
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    set -e

                    echo "Checking Nginx..."
                    curl -f http://localhost

                    echo ""
                    echo "Checking FastAPI..."
                    curl -f http://localhost:8000/

                    echo ""
                    echo "Health check completed successfully"
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
