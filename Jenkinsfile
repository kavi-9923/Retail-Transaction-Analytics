
pipeline {
    agent any

    options {
        timestamps()
    }

    environment {
        IMAGE_NAME = 'retail-analytics-dashboard'
        CONTAINER_NAME = 'retail-analytics-dashboard'
        APP_PORT = '3000'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/kavi-9923/Retail-Transaction-Analytics.git'
            }
        }

        stage('Validate') {
            steps {
                sh '''
                    test -f app.py
                    test -f Dockerfile
                    test -f requirements.txt
                    python3 -m py_compile app.py
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                      -t ${IMAGE_NAME}:${BUILD_NUMBER} \
                      -t ${IMAGE_NAME}:latest .
                '''
            }
        }

        stage('Smoke Test') {
            steps {
                sh '''
                    TEST_PORT=$(shuf -i 20000-30000 -n 1)

                    docker run -d --rm \
                      --name retail-test-${BUILD_NUMBER} \
                      -p 127.0.0.1:${TEST_PORT}:3000 \
                      ${IMAGE_NAME}:${BUILD_NUMBER}

                    trap 'docker stop retail-test-${BUILD_NUMBER} >/dev/null 2>&1 || true' EXIT

                    for i in $(seq 1 20); do
                        if curl -fsS http://127.0.0.1:${TEST_PORT}/; then
                            echo "Smoke test passed"
                            exit 0
                        fi
                        sleep 2
                    done

                    echo "Smoke test failed"
                    docker logs retail-test-${BUILD_NUMBER}
                    exit 1
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker rm -f ${CONTAINER_NAME} 2>/dev/null || true

                    docker run -d \
                      --name ${CONTAINER_NAME} \
                      --restart unless-stopped \
                      -p ${APP_PORT}:3000 \
                      ${IMAGE_NAME}:${BUILD_NUMBER}

                    sleep 3
                    curl -fsS http://127.0.0.1:${APP_PORT}/ >/dev/null

                    echo "Deployment successful"
                '''
            }
        }
    }

    post {
        success {
            echo 'Retail Analytics dashboard deployed successfully.'
        }
        failure {
            echo 'Pipeline failed. Review the stage logs in Console Output.'
        }
    }
}

