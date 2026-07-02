pipeline {
    agent any

    stages {

        stage('Checkout Service Repo') {
            steps {
                git url: 'https://github.com/wyidongh/cpp-demo-service.git', branch: 'main'
            }
        }

        stage('Build in Docker CI Image') {
            steps {
                sh '''
                docker run --rm \
                  -v $WORKSPACE:/workspace \
                  cpp-ci:build-1.0 \
                  bash -c "
                    cd /workspace &&
                    rm -rf build &&
                    mkdir build &&
                    cd build &&
                    cmake .. &&
                    make
                  "
                '''
            }
        }

        stage('Run App') {
            steps {
                sh './build/app'
            }
        }
    }
}
