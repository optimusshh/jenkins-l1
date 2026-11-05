pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Repository successfully checked out'
                sh '''
                    echo "Repository files:"
                    ls -la
                    echo "Git commit:"
                    git rev-parse --short HEAD
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    echo "Building application..."
                    mkdir -p build
                    echo "Hello from GitHub and Jenkins via github manual stage" > build/index.html
                    echo "Build completed"
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    echo "Testing application..."
                    test -f build/index.html
                    grep -q "Hello from GitHub and Jenkins via github manual stage" build/index.html
                    echo "All tests passed!"
                '''
            }
        }
    }
}
