pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "yces-python-app"
        VENV_PATH = "venv"
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
                sh '''
                    echo "[*] Creating virtual environment..."
                    python3 -m venv ${VENV_PATH}
                    . ${VENV_PATH}/bin/activate
                    echo "[*] Upgrading pip..."
                    pip install --upgrade pip
                    
                    if [ -f ${REQUIREMENTS_PATH} ]; then
                        echo "[*] Installing dependencies from ${REQUIREMENTS_PATH}..."
                        pip install -r ${REQUIREMENTS_PATH}
                    else
                        echo "[!] requirements.txt not found at ${REQUIREMENTS_PATH}"
                        exit 1
                    fi
                '''
            }
        }

        stage('Unit Tests') {
            steps {
                sh '''
                    . ${VENV_PATH}/bin/activate
                    if ls test_*.py >/dev/null 2>&1; then
                        echo "[*] Running tests..."
                        pytest
                    else
                        echo "[*] No tests found. Skipping."
                    fi
                '''
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
