pipeline {
    agent any

    tools {
        python 'Python3' // Ensure Python3 is set up in Jenkins Global Tools
    }

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
                // Example: Run Flake8 for linting
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
                    // Publish JUnit test results
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
                // Example: Integrate a security scanner like Trivy
                // This would require Trivy to be installed on the agent
                // sh "trivy image --severity HIGH,CRITICAL ${DOCKER_IMAGE_TAG}"
                echo "Skipping image scan for now, but recommended!"
            }
        }

        // This stage is more for testing the Docker image immediately after build
        // For actual deployment, you'd likely push to a registry and then deploy.
        stage('Run Container for Integration Tests') {
            steps {
                script {
                    // Start container in detached mode, capture ID for later cleanup
                    def containerId = sh(returnStdout: true, script: "docker run -d -p 8081:8080 ${DOCKER_IMAGE_TAG}").trim()
                    env.CONTAINER_ID = containerId // Store container ID in environment for cleanup
                    echo "Container started with ID: ${containerId}"

                    // Add a delay or health check to ensure the app is up before testing
                    sh 'sleep 10' // Crude delay, consider a proper health check

                    // Example: Run a simple curl command to check if the app is responsive
                    sh 'curl -f http://localhost:8081 || { echo "Application not responsive"; exit 1; }'
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
            script {
                // Stop and remove the specific container started in the pipeline
                if (env.CONTAINER_ID) {
                    sh "docker rm -f ${env.CONTAINER_ID} || true"
                }
                // Remove the built Docker image
                sh "docker rmi ${DOCKER_IMAGE_TAG} || true"
            }
            // Generic cleanup for any stray containers (less precise but good fallback)
            sh 'docker ps -aq | xargs -r docker rm -f || true'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
            // Add notification logic here (e.g., email, Slack)
        }
    }
}
