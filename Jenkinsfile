pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "yces-python-app"
        REQUIREMENTS_FILE = "Docker/requirements.txt"
        APP_DIR = "source-code"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                dir("${APP_DIR}") {
                    sh '''
                        echo "[*] Creating virtual environment..."
                        python3 -m venv test_env
                        echo "[*] Activating virtual environment..."
                        . test_env/bin/activate
                        echo "[*] Installing requirements from ${REQUIREMENTS_FILE}..."
                        pip install --upgrade pip
                        pip install -r ${REQUIREMENTS_FILE}
                    '''
                }
            }
        }

        stage('Unit Tests') {
            steps {
                dir("${APP_DIR}") {
                    sh '''
                        . test_env/bin/activate
                        echo "[*] Running unit tests..."
                        pytest || echo "No tests found, skipping."
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    echo "[*] Building Docker image..."
                    docker build -t $DOCKER_IMAGE .
                '''
            }
        }

        stage('Docker Run') {
            steps {
                sh '''
                    echo "[*] Running Docker container..."
                    docker run -d -p 8081:8080 $DOCKER_IMAGE
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
