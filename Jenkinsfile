pipeline {
    agent any

    stages {

	stage('Checkout Service Repo') {
	    steps {
		dir('service') {
		    git branch: 'main',
			url: 'https://github.com/wyidongh/cpp-demo-service.git'
		}
	    }
	}

	stage('Build') {
	    steps {
		sh '''
		docker run --rm \
		  -v $WORKSPACE/service:/workspace \
		  cpp-ci:build-1.0 \
		  bash -c "
		    mkdir -p build &&
		    cd build &&
		    cmake .. &&
		    make
		  "
		'''
	    }
	}
	
	stage('Run App') {
	    steps {
		sh '''
		docker run --rm \
		  -v $WORKSPACE/service:/workspace \
		  cpp-ci:build-1.0 \
		  bash -c "
		    ./build/app
		  "
		'''
	    }
	}

    }
}
