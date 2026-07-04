pipeline {

    parameters {
    choice(name: 'ENV', choices: ['dev', 'staging', 'prod'], description: 'Deploy Environment')
}
    agent any

    environment {
        WORKSPACE_DIR = "${WORKSPACE}"
        SERVICE_DIR   = "${WORKSPACE}/service"
        BUILD_DIR     = "${WORKSPACE}/build"
        IMAGE_TAG     = "cpp-demo-build:${BUILD_NUMBER}"
	IMAGE_NAME = "localhost:5000/cpp-demo"
	ENV_PORT_MAP = "dev:8081,staging:8082,prod:8083"	
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

	stage('Push Image') {
	    steps {
		sh """
		    docker tag ${IMAGE_TAG} localhost:5000/cpp-demo:${BUILD_NUMBER}
		    docker push localhost:5000/cpp-demo:${BUILD_NUMBER}
		"""
	    }
	}
	stage('Resolve Deploy Config') {
	    steps {
		script {
		    def map = [:]
		    env.ENV_PORT_MAP.split(',').each {
			def parts = it.split(':')
			map[parts[0]] = parts[1]
		    }

		    env.DEPLOY_PORT = map[params.ENV]
		    echo "Deploy ENV = ${params.ENV}, PORT = ${env.DEPLOY_PORT}"
		}
	    }
	}

        stage('Resolve Deploy Config') {
            // ... 已有代码 ...
        }

        stage('Deploy') {
            steps {
                sh """
                    set -e
                    docker rm -f cpp-demo-${params.ENV} || true
                    docker run -d --rm \
                        --name cpp-demo-${params.ENV} \
                        -p ${env.DEPLOY_PORT}:8080 \
                        localhost:5000/cpp-demo:${BUILD_NUMBER} \
                        sh -c 'echo "Running in ${params.ENV} on port ${env.DEPLOY_PORT}"; sleep 3600'
                    echo "Deployed cpp-demo-${params.ENV} on port ${env.DEPLOY_PORT}"
                """
            }
        }


    post {
        always {
            sh '''
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
