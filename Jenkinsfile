pipeline {
    agent any

    tools {
        python 'Python3'  // Make sure 'Python3' is configured in Jenkins global tools
    }

    environment {
        DOCKER_IMAGE = "yces-python-app"
        REQUIREMENTS_PATH = "source-code/Docker/requirements.txt"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('source-code') {
                    sh '''
                        echo "[*] Creating virtual environment..."
                        python3 -m venv venv
                        source venv/bin/activate
                        pip install --upgrade pip
                        pip install -r Docker/requirements.txt
                    '''
                }
            }
        }

        stage('Unit Tests') {
            steps {
                dir('source-code') {
                    sh '''
                        source venv/bin/activate
                        if ls test_*.py >/dev/null 2>&1; then
                            echo "[*] Running tests..."
                            pytest
                        else
                            echo "[*] No tests found. Skipping."
                        fi
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Docker Run') {
            steps {
                sh 'docker run -d -p 8081:8080 $DOCKER_IMAGE'
            }
        }
    }

    post {
        always {
            echo 'Cleaning up containers if any...'
            sh 'docker ps -aq | xargs -r docker rm -f || true'
        }
    }
}
