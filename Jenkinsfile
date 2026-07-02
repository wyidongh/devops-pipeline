pipeline {
    agent any

    environment {
        SERVICE_NAME = "cpp-demo-service"
        SERVICE_PATH = "${WORKSPACE}/service"
        IMAGE_NAME = "cpp-demo:${BUILD_NUMBER}"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                sh '''
                    echo "Cleaning workspace..."
                    rm -rf ${SERVICE_PATH} || true
                '''
            }
        }

        stage('Checkout') {
            steps {
                sh '''
                    git clone https://github.com/wyidongh/cpp-demo-service.git ${SERVICE_PATH}
                '''
            }
        }

        stage('Build (Isolated Docker)') {
            steps {
                sh '''
                    docker run --rm \
                      -v ${SERVICE_PATH}:/src \
                      -u $(id -u):$(id -g) \
                      cpp-ci:build-1.0 \
                      bash -c "
                        set -e
                        cd /src

                        rm -rf build
                        mkdir build
                        cd build

                        cmake ..
                        make
                      "
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    docker run --rm \
                      -v ${SERVICE_PATH}:/src \
                      -u $(id -u):$(id -g) \
                      cpp-ci:build-1.0 \
                      bash -c "
                        cd /src/build
                        if [ -f CTestTestfile.cmake ]; then
                            ctest --output-on-failure
                        else
                            echo 'No tests found, skipping'
                        fi
                      "
                '''
            }
        }

        stage('Archive Artifact') {
            steps {
                sh '''
                    mkdir -p ${WORKSPACE}/artifact
                    cp ${SERVICE_PATH}/build/app ${WORKSPACE}/artifact/
                '''
                archiveArtifacts artifacts: 'artifact/app'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
cat > ${SERVICE_PATH}/Dockerfile <<EOF
FROM ubuntu:22.04
WORKDIR /app
COPY build/app /app/app
RUN chmod +x /app/app
CMD ["/app/app"]
EOF

docker build -t ${IMAGE_NAME} ${SERVICE_PATH}
                '''
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    docker run --rm ${IMAGE_NAME}
                '''
            }
        }

    }

    post {
        success {
            echo "CI/CD SUCCESS 🎉"
        }
        failure {
            echo "CI FAILED ❌"
        }
        always {
            sh 'echo "cleanup done"'
        }
    }
}
