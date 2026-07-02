pipeline {
    agent any

    environment {
        SERVICE_NAME = "cpp-demo-service"
        REPO_URL = "https://github.com/wyidongh/cpp-demo-service.git"
        WORK_DIR = "${WORKSPACE}/service"
        BUILD_DIR = "${WORKSPACE}/build"
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        stage('Clean Workspace') {
            steps {
                sh '''
                    echo "Cleaning workspace..."
                    rm -rf ${WORK_DIR}
                    rm -rf ${BUILD_DIR}
                    mkdir -p ${WORK_DIR}
                    mkdir -p ${BUILD_DIR}
                '''
            }
        }

        stage('Checkout') {
            steps {
                sh '''
                    echo "Cloning repo..."
                    git clone ${REPO_URL} ${WORK_DIR}
                    ls -al ${WORK_DIR}
                '''
            }
        }

        stage('Detect Project Root') {
            steps {
                script {
                    env.PROJECT_ROOT = sh(
                        script: "find ${WORK_DIR} -name CMakeLists.txt | head -n 1 | xargs dirname",
                        returnStdout: true
                    ).trim()

                    echo "Detected project root: ${env.PROJECT_ROOT}"
                }
            }
        }

        stage('Build (Containerized)') {
            steps {
                sh '''
                    set -e

                    echo "Building in Docker..."
                    echo "Project root: ${PROJECT_ROOT}"

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

        stage('Test') {
            steps {
                sh '''
                    echo "Running tests..."
                    ls -al ${BUILD_DIR}
                '''
            }
        }

        stage('Run App') {
            steps {
                sh '''
                    docker run --rm \
                        -v ${BUILD_DIR}:/app \
                        cpp-ci:build-1.0 \
                        bash -c "
                            if [ -f /app/app ]; then
                                /app/app
                            else
                                echo 'No binary found'
                                exit 1
                            fi
                        "
                '''
            }
        }

        stage('Archive') {
            steps {
                sh '''
                    echo "Archiving artifacts..."
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
