pipeline {
    agent any

    environment {
        SERVICE_PATH = "/home/dong/devops/workspace/cpp-demo-service"
        IMAGE_NAME = "cpp-app:${BUILD_NUMBER}"
    }

    stages {

        stage('Prepare') {
            steps {
                sh 'rm -rf ${SERVICE_PATH}'
            }
        }

        stage('Checkout') {
            steps {
                sh '''
                    git clone https://github.com/wyidongh/cpp-demo-service.git ${SERVICE_PATH}
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    docker run --rm -v ${SERVICE_PATH}:/src cpp-ci:build-1.0 bash -c "
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
                    docker run --rm -v ${SERVICE_PATH}:/src cpp-ci:build-1.0 bash -c "
                        cd /src/build
                        ctest --output-on-failure || true
                    "
                '''
            }
        }

        stage('Package Docker Image') {
            steps {
                sh '''
                    cat > ${SERVICE_PATH}/Dockerfile <<EOF
FROM ubuntu:22.04
WORKDIR /app
COPY build/app .
CMD ["./app"]
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
    }
}
