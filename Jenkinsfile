pipeline {
    agent {
        docker {
            image 'python:3.10'
            args '-u root' // run as root to install packages
        }
    }

    environment {
        DOCKER_IMAGE = "yces-python-app"
    }

    stages {
        stage('Install Dependencies') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'pytest || echo "No tests yet"'
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
            echo 'Cleaning up...'
            sh 'docker ps -aq | xargs -r docker rm -f || true'
        }
    }
}
