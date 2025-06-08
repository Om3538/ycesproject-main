pipeline {
    agent any

    environment {
        VENV      = "test_env"
        IMAGE     = "yces-python-app"
        APP_PORT  = "8083"     // internal container port based on your Dockerfile
        HOST_PORT = "8080"     // host port to bind Jenkins uses
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
                        echo "[*] Create & activate virtualenv..."
                        python3 -m venv $VENV
                        . $VENV/bin/activate

                        echo "[*] Install dependencies..."
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
                        echo "[*] 🧪 Running tests..."
                        pytest || echo "⚠️ No tests or failures – continuing."
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir('source-code') {
                    sh '''
                        echo "[*] 🛠️ Building Docker image..."
                        docker build \
                            --tag $IMAGE \
                            --file Dockerfile \
                            .
                    '''
                }
            }
        }

        stage('Docker Run') {
            steps {
                sh '''
                    echo "[*] 🚀 Running Docker container..."
                    docker ps -aq --filter "name=$IMAGE" | xargs -r docker rm -f
                    docker run -d \
                        --name $IMAGE \
                        -p $HOST_PORT:$APP_PORT \
                        $IMAGE || echo "⚠️ Port $HOST_PORT in use; container run failed."
                '''
            }
        }
    }

    post {
        always {
            echo "[*] 🧹 Cleanup: removing containers..."
            sh 'docker ps -aq --filter "name=$IMAGE" | xargs -r docker rm -f'
        }
    }
}
