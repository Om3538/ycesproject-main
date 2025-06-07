pipeline {
    agent any

    environment {
        PROJECT_DIR = 'source-code'
        IMAGE_NAME = 'ycesproject-image'
        CONTAINER_NAME = 'ycesproject-container'
        PORT = '8083'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Om3538/ycesproject-main.git'
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
                        echo "Running Unit Tests (placeholder)"
                        # Add real test command if available
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir("${PROJECT_DIR}") {
                    sh """
                        docker build -t ${IMAGE_NAME} .
                    """
                }
            }
        }

        stage('Docker Run') {
            steps {
                sh """
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                    docker run -d -p 80:${PORT} --name ${CONTAINER_NAME} ${IMAGE_NAME}
                """
            }
        }
    }

    post {
        always {
            echo 'Cleaning up containers if any...'
            sh 'docker ps -aq | xargs -r docker rm -f || true'
        }
        failure {
            echo 'Pipeline Failed ❌'
        }
        success {
            echo 'Pipeline Completed Successfully ✅'
        }
    }
}
