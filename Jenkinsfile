pipeline {
    agent any

    environment {
        VENV = "test_env"
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
                        echo [*] Creating virtual environment...
                        python3 -m venv $VENV
                        echo [*] Activating virtual environment...
                        . $VENV/bin/activate
                        echo [*] Installing requirements from Docker/requirements.txt...
                        pip install --upgrade pip
                        pip install -r Docker/requirements.txt
                        pip install pytest
                    '''
                }
            }
        }

        stage('Unit Tests') {
            steps {
                dir('source-code') {
                    sh '''
                        . $VENV/bin/activate
                        echo [*] Running unit tests...
                        if pytest; then
                            echo "Tests passed."
                        else
                            echo "Tests failed."
                            exit 1
                        fi
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir('source-code') {
                    sh '''
                        echo [*] Building Docker image...
                        docker build -t yces-python-app Docker/
                    '''
                }
            }
        }

        stage('Docker Run') {
            steps {
                sh '''
                    echo [*] Running Docker container...
                    docker run -d -p 8080:8080 --name yces-python-app yces-python-app
                '''
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
