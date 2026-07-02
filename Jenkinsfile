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

			echo '===== PROJECT STRUCTURE ====='
			find . -name CMakeLists.txt

			echo '===== FIND ROOT ====='
			ROOT_DIR=$(dirname $(find . -name CMakeLists.txt | head -n 1))

			echo 'ROOT_DIR:' $ROOT_DIR

			cd $ROOT_DIR

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
