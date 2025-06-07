pipeline {
    agent {
        docker {
            image 'python:3.10'
        }
    }

    environment {
        VENV_DIR = 'venv'
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
                    python3 -m venv ${VENV_DIR}
                    . ${VENV_DIR}/bin/activate
                    pip install --upgrade pip
                    pip install -r source-code/requirements.txt
                '''
            }
        }

        stage('Unit Tests') {
            steps {
                sh '''
                    . ${VENV_DIR}/bin/activate
                    # run your tests here (update command if needed)
                    echo "No unit tests defined."
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t ycesproject:latest .
                '''
            }
        }

        stage('Docker Run') {
            steps {
                sh '''
                    docker stop ycesproject || true
                    docker rm ycesproject || true
                    docker run -d --name ycesproject -p 8083:80 ycesproject:latest
                '''
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
            sh 'docker ps -aq | xargs -r docker rm -f || true'
        }
    }
}
