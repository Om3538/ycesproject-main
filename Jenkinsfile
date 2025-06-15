pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = "yces-python-app"
        DOCKER_IMAGE_TAG = "${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}"
        // Add other environment variables as needed, e.g., for registry credentials
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'python3 -m venv venv'
                sh '. venv/bin/activate && pip install -r requirements.txt'
            }
        }

        stage('Code Linting') {
            steps {
                sh '. venv/bin/activate && pip install flake8' // Install linter
                sh '. venv/bin/activate && flake8 --exit-zero || true' // Exit-zero allows pipeline to continue on warnings
            }
        }

        stage('Unit Tests') {
            steps {
                sh '. venv/bin/activate && pytest --junitxml=reports/test-results.xml || echo "No tests yet or tests failed"'
            }
            post {
                always {
                    junit 'reports/test-results.xml'
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    echo "Building Docker image: ${DOCKER_IMAGE_TAG}"
                    sh "docker build -t ${DOCKER_IMAGE_TAG} ."
                }
            }
        }

        stage('Docker Image Scan') {
            steps {
                echo "Skipping image scan for now, but recommended!"
            }
        }

        stage('Run Container for Integration Tests') {
            steps {
                script {
                    def containerId = sh(returnStdout: true, script: "docker run -d -p 8081:8080 ${DOCKER_IMAGE_TAG}").trim()
                    env.CONTAINER_ID = containerId
                    echo "Container started with ID: ${containerId}"

                    sh 'sleep 10'
                    sh 'curl -f http://localhost:8081 || { echo "Application not responsive"; exit 1; }'
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
            script {
                if (env.CONTAINER_ID) {
                    sh "docker rm -f ${env.CONTAINER_ID} || true"
                }
                sh "docker rmi ${DOCKER_IMAGE_TAG} || true"
            }
            sh 'docker ps -aq | xargs -r docker rm -f || true'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
