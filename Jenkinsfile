
pipeline {
    agent {
        label 'linux-agent'
    }

    stages {

        // --------------------------------
        // 1. CHECKOUT
        // --------------------------------

        stage('Checkout') {
            steps {
                echo 'Repository automatically checked out using SCM'

                sh '''
                    echo "Repository files:"
                    ls -la

                    echo "Git commit:"
                    git rev-parse --short HEAD
                '''
            }
        }


        // --------------------------------
        // 2. BUILD
        // --------------------------------

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


        // --------------------------------
        // 3. TEST
        // --------------------------------

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


        // --------------------------------
        // 4. BACKUP
        // --------------------------------

        stage('Backup') {
            steps {
                echo 'Creating datewise backup on VM3'

                sh '''
                    ssh -i /home/jenkins-agent/.ssh/id_ed25519 \
                        -o BatchMode=yes \
                        deploy@192.168.31.190 \
                        'set -eu

                        if [ -f /opt/myapp/index.html ]; then

                            BACKUP_FILE="/opt/myapp-backups/index-$(date +%Y%m%d-%H%M%S-%N).html"

                            cp -a /opt/myapp/index.html "$BACKUP_FILE"

                            echo "Backup created successfully:"
                            echo "$BACKUP_FILE"

                        else
                            echo "No existing application found. Skipping backup."
                        fi'
                '''
            }
        }


        // --------------------------------
        // 5. DEPLOY
        // --------------------------------

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


        // --------------------------------
        // 6. VERIFY
        // --------------------------------

        stage('Verify') {
            steps {
                sh '''
                    echo "Checking application availability..."

                    HTTP_CODE=$(curl -sS -o /dev/null -w "%{http_code}" http://192.168.31.190/)

                    echo "HTTP Status: $HTTP_CODE"

                    test "$HTTP_CODE" = "200"

                    echo "Application is responding successfully!"
                '''
            }
        }

    }
}
