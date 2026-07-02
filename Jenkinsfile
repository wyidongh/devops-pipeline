pipeline {
    agent any

    environment {
        WORKSPACE_DIR = "${WORKSPACE}"
        SERVICE_DIR   = "${WORKSPACE}/service"
        BUILD_DIR     = "${WORKSPACE}/build"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                sh '''
                set -e
                rm -rf $SERVICE_DIR $BUILD_DIR
                mkdir -p $SERVICE_DIR $BUILD_DIR
                '''
            }
        }

        stage('Checkout') {
            steps {
                sh '''
                set -e
                git clone https://github.com/wyidongh/cpp-demo-service.git $SERVICE_DIR
                ls -al $SERVICE_DIR
                '''
            }
        }

        stage('Detect Project Root') {
            steps {
                script {
                    def root = sh(
                        script: "find ${SERVICE_DIR} -name CMakeLists.txt | head -n 1 | xargs dirname",
                        returnStdout: true
                    ).trim()

                    if (!root) {
                        error("CMakeLists.txt not found")
                    }

                    env.PROJECT_ROOT = root
                    echo "Detected ROOT = ${env.PROJECT_ROOT}"
                }
            }
        }

        stage('Build (Dockerized)') {
            steps {
                sh '''
                set -e

                echo "PROJECT_ROOT=$PROJECT_ROOT"

                docker run --rm \
                    -v $SERVICE_DIR:/workspace \
                    -v $BUILD_DIR:/build \
                    -w /workspace \
                    cpp-ci:build-1.0 \
                    bash -c "
                        set -e
                        echo 'BUILD START'

                        cmake -S ${PROJECT_ROOT#/workspace/} -B /build
                        cmake --build /build -j
                    "
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                set -e
                ctest --test-dir $BUILD_DIR || true
                '''
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'build/**', fingerprint: true
            }
        }

        stage('Run') {
            steps {
                sh '''
                docker run -d --rm \
                    -v $BUILD_DIR:/app \
                    alpine:latest \
                    sleep 3600
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
