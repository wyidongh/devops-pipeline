pipeline {
    agent any

    environment {
        SERVICE_DIR = "${WORKSPACE}/service"
        IMAGE = "cpp-ci:build-1.0"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                sh '''
                    echo "Cleaning workspace..."
                    rm -rf ${SERVICE_DIR}
                    mkdir -p ${SERVICE_DIR}
                '''
            }
        }

        stage('Checkout') {
            steps {
                sh '''
                    git clone https://github.com/wyidongh/cpp-demo-service.git ${SERVICE_DIR}
                '''
            }
        }

	stage('Build (Container Isolated)') {
	    steps {
		sh '''
		    docker run --rm \
		      -v ${SERVICE_DIR}:/src \
		      -w /src \
		      cpp-ci:build-1.0 \
		      bash -c "
			set -e

			echo '===== FIND PROJECT ROOT ====='
			ROOT=$(dirname $(find /src -name CMakeLists.txt | head -n 1))

			echo 'Project root:' $ROOT

			cd $ROOT

			rm -rf build
			cmake -S . -B build
			cmake --build build -j
		      "
		'''
	    }
	}

        stage('Test') {
            steps {
                sh '''
                    echo "No tests yet"
                '''
            }
        }

        stage('Archive') {
            steps {
                sh '''
                    mkdir -p ${WORKSPACE}/artifacts
                    cp ${SERVICE_DIR}/build/app ${WORKSPACE}/artifacts/
                '''
            }
        }

        stage('Run') {
            steps {
                sh '''
                    docker run --rm \
                      -v ${SERVICE_DIR}/build:/app \
                      alpine \
                      /app/app
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
