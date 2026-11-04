pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo 'Source code is available'
            }
        }

        stage('Build') {
            steps {
                sh '''
                    echo "Building application..."
                    mkdir -p build
                    echo "Hello from GitHub and Jenkins" > build/index.html
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    echo "Testing application..."
                    test -f build/index.html
                    cat build/index.html
                '''
            }
        }

    }
}

