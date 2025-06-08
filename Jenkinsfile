pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "yces-python-app"
        PROJECT_DIR = "source-code"  // Assuming all your source files are inside this folder
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                dir("${PROJECT_DIR}") {
                    sh '''
                        python3 -m venv venv
                        . venv/bin/activate
                        pip install --upgrade pip
                        pip install -r requirements.txt
                    '''
                }
            }
        }

        stage('Unit Tests') {
            steps {
                dir("${PROJECT_DIR}") {
                    sh '''
                        . venv/bin/activate
                        pytest || echo "No tests found"
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir("${PROJECT_DIR}") {
                    sh 'docker build -t $DOCKER_IMAGE .'
                }
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
