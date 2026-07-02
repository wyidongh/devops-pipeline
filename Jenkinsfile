pipeline {
    agent any

    environment {
        REPO_URL = "https://github.com/wyidongh/cpp-demo-service.git"
        WORK_DIR = "${WORKSPACE}/service"
        BUILD_DIR = "${WORKSPACE}/build"
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        stage('Clean') {
            steps {
                sh '''
                    rm -rf ${WORK_DIR} ${BUILD_DIR}
                    mkdir -p ${WORK_DIR} ${BUILD_DIR}
                '''
            }
        }

        stage('Checkout') {
            steps {
                sh '''
                    git clone ${REPO_URL} ${WORK_DIR}
                    ls -al ${WORK_DIR}
                '''
            }
        }

        stage('Detect Root') {
            steps {
                script {
                    env.PROJECT_ROOT = sh(
                        script: "dirname $(find ${WORK_DIR} -name CMakeLists.txt | head -n 1)",
                        returnStdout: true
                    ).trim()

                    echo "PROJECT_ROOT = ${env.PROJECT_ROOT}"
                }
            }
        }

        stage('Build') {
            steps {
                sh '''
                    set -e

                    echo "Building project..."

                    docker run --rm \
                        -v ${WORK_DIR}:/workspace \
                        -v ${BUILD_DIR}:/build \
                        -w ${PROJECT_ROOT} \
                        cpp-ci:build-1.0 \
                        bash -c "
                            set -e
                            cmake -S . -B /build
                            cmake --build /build -j
                        "
                '''
            }
        }

        stage('Run') {
            steps {
                sh '''
                    docker run --rm \
                        -v ${BUILD_DIR}:/build \
                        cpp-ci:build-1.0 \
                        bash -c "
                            if [ -f /build/app ]; then
                                /build/app
                            else
                                echo 'binary not found'
                                ls -al /build
                                exit 1
                            fi
                        "
                '''
            }
        }

        stage('Archive') {
            steps {
                sh '''
                    mkdir -p ${WORKSPACE}/artifacts
                    cp -f ${BUILD_DIR}/app ${WORKSPACE}/artifacts/ || true
                '''
            }
        }
    }

    post {
        success {
            echo "CI SUCCESS ✅"
        }
        failure {
            echo "CI FAILED ❌"
        }
    }
}
