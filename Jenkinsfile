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
		rm -rf service build

		git clone https://github.com/wyidongh/cpp-demo-service.git service
		ls -al service
		'''
	    }
	}

        stage('Detect Project Root') {
            steps {
                script {
                	env.PROJECT_ROOT = "${WORKSPACE}/service"
			echo "PROJECT_ROOT = ${env.PROJECT_ROOT}"
		}
            }
        }

	stage('Build') {
	    steps {
		sh '''
		set -e

		docker run --rm \
		  -v $WORKSPACE/service:/workspace/service \
		  -v $WORKSPACE/build:/build \
		  -w /workspace/service cpp-ci:build-1.0 bash -c "
		    set -e
		    ls -al

		    cd cpp-demo-service

		    echo '=== BUILD IN:' $(pwd)
		    ls -al

		    cmake -S . -B /build
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
