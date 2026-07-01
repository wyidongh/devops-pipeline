pipeline {
    agent any

    stages {

        stage('Checkout Service Repo') {
            steps {
                echo "Cloning cpp-demo-service..."
                git branch: 'main',
                    url: 'https://github.com/wyidongh/cpp-demo-service.git'
            }
        }

        stage('Verify Workspace') {
            steps {
                sh 'ls -al'
                sh 'echo "BUILD SUCCESS - Checkout Only"'
            }
        }
    }
}
