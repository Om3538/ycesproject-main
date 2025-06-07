pipeline {
    agent any

    stages {
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
                echo 'No unit tests configured yet.'
            }
        }

        stage('Docker Build') {
            steps {
                dir('source-code') {
                    sh 'docker build -t ycesproject .'
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
            echo 'Cleaning up containers if any...'
            sh 'docker ps -aq | xargs -r docker rm -f || true'
        }
    }
}
