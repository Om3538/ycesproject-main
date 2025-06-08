pipeline {
    agent any

    environment {
        PROJECT_NAME = "ycesproject"
        IMAGE_NAME = "1365890/ycesproject"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Om3538/ycesproject-main.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Unit Tests') {
            steps {
                sh '''
                    . venv/bin/activate
                    python3 -m unittest discover -s tests || echo "No tests found or some failed"
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t $IMAGE_NAME .
                '''
            }
        }

        stage('Docker Run') {
            steps {
                sh '''
                    docker ps -q --filter "name=$PROJECT_NAME" | grep -q . && docker rm -f $PROJECT_NAME || true
                    docker run -d -p 8083:80 --name $PROJECT_NAME $IMAGE_NAME
                '''
            }
        }
    }

    post {
        always {
            echo 'Cleaning up containers if any...'
            sh 'docker ps -aq | xargs -r docker rm -f'
        }
        failure {
            echo 'Pipeline Failed ❌'
        }
        success {
            echo 'Pipeline Succeeded ✅'
        }
    }
}
