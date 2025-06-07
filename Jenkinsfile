pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/Om3538/ycesproject-main.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('source-code') {
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
                echo 'Skipping tests (add here if needed)'
            }
        }

        stage('Docker Build') {
            steps {
                dir('source-code') {
                    sh '''
                        docker build -t ycesproject .
                    '''
                }
            }
        }

        stage('Docker Run') {
            steps {
                sh '''
                    docker stop ycesproject || true
                    docker rm ycesproject || true
                    docker run -d -p 8083:80 --name ycesproject ycesproject
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
