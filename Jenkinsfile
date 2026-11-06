
pipeline {
    agent {
        label 'linux-agent'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Repository automatically checked out using Poll SCM'

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

        stage('Deploy') {
            steps {
                echo 'Deploying application to VM3'

                sh '''
                    scp -i /home/jenkins-agent/.ssh/id_ed25519 \
                        -o BatchMode=yes \
                        build/index.html \
                        deploy@192.168.31.190:/opt/myapp/

                    echo "Deployment completed"

                    ssh -i /home/jenkins-agent/.ssh/id_ed25519 \
                        -o BatchMode=yes \
                        deploy@192.168.31.190 \
                        'hostname && whoami && cat /opt/myapp/index.html'
                '''
            }
        }
    }
}
