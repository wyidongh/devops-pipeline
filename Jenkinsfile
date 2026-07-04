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

        stage('Format Check') {
            steps {
                sh '''
                set -e
                cat > Dockerfile.format << 'EOF'
FROM cpp-ci:build-2.0
COPY service/ /workspace/
WORKDIR /workspace
RUN clang-format --dry-run --Werror main.cpp tests/test_main.cpp
EOF
                docker build -f Dockerfile.format -t cpp-demo-format:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Static Analysis') {
            steps {
                sh '''
                set -e
                cat > Dockerfile.analysis << 'EOF'
FROM cpp-ci:build-2.0
COPY service/ /workspace/
WORKDIR /workspace
RUN cppcheck \
    --enable=all \
    --inconclusive \
    --std=c++17 \
    --language=c++ \
    .
EOF
                docker build -f Dockerfile.analysis -t cpp-demo-analysis:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Build') {
            steps {
                script {
                    sh '''
                        set -e
                        cat > Dockerfile.build << 'EOF'
FROM cpp-ci:build-2.0
COPY service/ /workspace/
WORKDIR /workspace
RUN cmake -S . -B /build \
    -DCMAKE_BUILD_TYPE=Debug \
    -DCMAKE_CXX_FLAGS="-g -O1 --coverage -fsanitize=address,undefined -fno-omit-frame-pointer" \
    -DCMAKE_C_FLAGS="-g -O1 --coverage -fsanitize=address,undefined -fno-omit-frame-pointer" \
 && cmake --build /build -j
EOF
                        docker build -f Dockerfile.build -t ${IMAGE_TAG} .
                        
                        docker create --name cpp-demo-extract-${BUILD_NUMBER} ${IMAGE_TAG}
                        rm -rf ${BUILD_DIR}
                        docker cp cpp-demo-extract-${BUILD_NUMBER}:/build ${BUILD_DIR}
                        docker rm cpp-demo-extract-${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Coverage') {
            steps {
                sh '''
                set -e
                
                cat > Dockerfile.coverage << 'EOF'
FROM cpp-ci:build-2.0
COPY service/ /workspace/
WORKDIR /workspace
RUN cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug -DCMAKE_CXX_FLAGS='--coverage' -DCMAKE_C_FLAGS='--coverage' && cmake --build build && ctest --test-dir build && gcovr -r . --html --html-details -o coverage.html
EOF
                
                docker build -f Dockerfile.coverage -t cpp-demo-coverage:${BUILD_NUMBER} .
                
                docker create --name cpp-demo-coverage-extract-${BUILD_NUMBER} cpp-demo-coverage:${BUILD_NUMBER}
                docker cp cpp-demo-coverage-extract-${BUILD_NUMBER}:/workspace/coverage.html ${WORKSPACE}/coverage.html
                docker rm cpp-demo-coverage-extract-${BUILD_NUMBER}
                '''
                
                archiveArtifacts artifacts: 'coverage.html', fingerprint: true
            }
        }

	stage('Test (ASan)') {
	    steps {
		sh '''
		docker run --rm \
		  --cap-add=SYS_PTRACE \
		  --security-opt seccomp=unconfined \
		  ${IMAGE_TAG} \
		  bash -c "
		    set -e
		    ctest --test-dir /build --output-on-failure
		    echo '--- Running app with ASan ---'
		    /build/app
		  "
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
                docker rm -f cpp-demo-run-${BUILD_NUMBER} 2>/dev/null || true
                docker rm -f cpp-demo-coverage-extract-${BUILD_NUMBER} 2>/dev/null || true
                docker rmi -f ${IMAGE_TAG} 2>/dev/null || true
                docker rmi -f cpp-demo-format:${BUILD_NUMBER} 2>/dev/null || true
                docker rmi -f cpp-demo-analysis:${BUILD_NUMBER} 2>/dev/null || true
                docker rmi -f cpp-demo-coverage:${BUILD_NUMBER} 2>/dev/null || true
                rm -f Dockerfile.build Dockerfile.format Dockerfile.analysis Dockerfile.coverage
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
