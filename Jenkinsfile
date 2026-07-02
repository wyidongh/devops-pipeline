pipeline {
    agent any

    environment {
        WORKSPACE_DIR = "/home/dong/devops/workspace"
        SERVICE_NAME = "cpp-demo-service"
        SERVICE_PATH = "/home/dong/devops/workspace/cpp-demo-service"
    }

    stages {

        stage('Prepare') {
            steps {
                sh '''
                    rm -rf ${SERVICE_PATH}
                    mkdir -p ${WORKSPACE_DIR}
                '''
            }
        }

        stage('Clone') {
            steps {
                sh '''
                    git clone https://github.com/wyidongh/cpp-demo-service.git ${SERVICE_PATH}
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    ls -al ${SERVICE_PATH}
                '''
            }
        }

        stage('Build (Docker)') {
            steps {
                sh '''
                    docker run --rm \
                      -v ${SERVICE_PATH}:/src \
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

        stage('Run') {
            steps {
                sh '''
                    docker run --rm \
                      -v ${SERVICE_PATH}:/src \
                      cpp-ci:build-1.0 \
                      bash -c "
                        /src/build/app
                      "
                '''
            }
        }
    }
}
