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

		    echo "UID=$(id -u)"
		    echo "GID=$(id -g)"

		    docker run --rm \
		      -v ${SERVICE_DIR}:/workspace \
		      -w /workspace \
		      -u 0 \
		      cpp-ci:build-1.0 \
		      bash -c "
			set -e
			rm -rf build
			mkdir -p build
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
