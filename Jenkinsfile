pipeline {
    agent any

    environment {
        SERVICE_REPO = "https://github.com/wyidongh/cpp-demo-service.git"
        SERVICE_DIR = "${WORKSPACE}/service"
        BUILD_IMAGE = "cpp-ci:build-1.0"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                sh '''
                    echo "Cleaning workspace..."
                    rm -rf ${SERVICE_DIR}
                '''
            }
        }

        stage('Checkout') {
            steps {
                sh '''
                    git clone ${SERVICE_REPO} ${SERVICE_DIR}
                '''
            }
        }

	stage('Build (Docker Isolated)') {
	    steps {
		sh '''
		    set -e

		    docker run --rm \
		      -v ${SERVICE_DIR}:/workspace \
		      -w /workspace \
		      cpp-ci:build-1.0 \
		      bash -c "
			set -e

			echo '===== DEBUG FILE TREE ====='
			find . -maxdepth 3

			echo '===== LOCATE CMakeLists ====='
			find . -name CMakeLists.txt

			echo '===== BUILD ====='

			# 自动进入正确目录
			cd $(dirname $(find . -name CMakeLists.txt | head -n 1))

			rm -rf build
			cmake -S . -B build
			cmake --build build
		      "
		'''
	    }
	}

        stage('Test') {
            steps {
                sh '''
                    echo "Run tests here (optional)"
                    ls -al ${SERVICE_DIR}/build
                '''
            }
        }

        stage('Archive Artifact') {
            steps {
                sh '''
                    mkdir -p ${WORKSPACE}/artifacts
                    cp ${SERVICE_DIR}/build/app ${WORKSPACE}/artifacts/
                '''
            }
        }

        stage('Run App') {
            steps {
                sh '''
                    ${SERVICE_DIR}/build/app
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
