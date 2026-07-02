pipeline {
    agent any

    environment {
        SERVICE_DIR = "/workspace/cpp-demo-service"
    }

    stages {

        stage('Prepare Workspace') {
            steps {
                sh '''
                    rm -rf ${SERVICE_DIR}
                    mkdir -p /workspace
                '''
            }
        }

        stage('Clone Service') {
            steps {
                sh '''
                    git clone \
                    https://github.com/wyidongh/cpp-demo-service.git \
                    ${SERVICE_DIR}
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    pwd

                    ls -al /workspace

                    ls -al ${SERVICE_DIR}
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    docker run --rm \
                      -v ${SERVICE_DIR}:/workspace \
                      cpp-ci:build-1.0 \
                      bash -c "
                        mkdir build &&
                        cd build &&
                        cmake .. &&
                        make
                      "
                '''
            }
        }

        stage('Run') {
            steps {
                sh '''
                    docker run --rm \
                      -v ${SERVICE_DIR}:/workspace \
                      cpp-ci:build-1.0 \
                      bash -c "./build/app"
                '''
            }
        }
    }
}
