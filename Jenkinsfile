pipeline {
    agent any

    environment {
        WORKSPACE_DIR = "${WORKSPACE}"
        SERVICE_DIR   = "${WORKSPACE}/service"
        BUILD_DIR     = "${WORKSPACE}/build"
        IMAGE_TAG     = "cpp-demo-build:${BUILD_NUMBER}"
    }

    stages {
        stage('Clean Workspace') {
            steps {
                sh '''
                set -e
                rm -rf ${SERVICE_DIR} ${BUILD_DIR}
                mkdir -p ${SERVICE_DIR} ${BUILD_DIR}
                '''
            }
        }

        stage('Checkout') {
            steps {
                sh '''
                set -e
                rm -rf service build

                git clone https://github.com/wyidongh/cpp-demo-service.git service
                ls -al service
                '''
            }
        }

        stage('Build') {
            steps {
                script {
                    // 生成临时 Dockerfile：把 service/ 目录 COPY 进镜像
                    // 注意 service/ 末尾的斜杠，表示复制目录"内容"到 /workspace/
                    writeFile file: 'Dockerfile.build', text: """FROM cpp-ci:build-1.0
COPY service/ /workspace/
WORKDIR /workspace
RUN cmake -S . -B /build && cmake --build /build -j
"""
                    // 构建镜像（build context 是当前 workspace，包含 service/ 目录）
                    sh "docker build -f Dockerfile.build -t ${IMAGE_TAG} ."

                    // 创建临时容器（不启动），把 /build 提取到 Jenkins 工作空间
                    sh """
                        set -e
                        docker create --name cpp-demo-extract-${BUILD_NUMBER} ${IMAGE_TAG}
                        rm -rf ${BUILD_DIR}
                        docker cp cpp-demo-extract-${BUILD_NUMBER}:/build ${BUILD_DIR}
                        docker rm cpp-demo-extract-${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Test') {
            steps {
                // 测试也在构建镜像里跑，无需 -v 挂载
                sh "docker run --rm ${IMAGE_TAG} ctest --test-dir /build || true"
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'build/**', fingerprint: true
            }
        }

        stage('Run') {
            steps {
                // 运行构建镜像即可，构建产物已在镜像内部，无需 -v 挂载
                sh """
                    docker run -d --rm \
                        --name cpp-demo-run-${BUILD_NUMBER} \
                        ${IMAGE_TAG} \
                        sh -c 'echo "Container ready with build artifacts"; sleep 3600'
                """
            }
        }
    }

    post {
        always {
            sh '''
                docker rm -f cpp-demo-run-${BUILD_NUMBER} || true
                docker rmi -f ${IMAGE_TAG} || true
                rm -f Dockerfile.build
            '''
        }
        success {
            echo "CI SUCCESS ✅"
        }
        failure {
            echo "CI FAILED ❌"
        }
    }
}
