
pipeline {
    agent {
        label 'linux-agent'
    }

    stages {

        // 1. CHECKOUT
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


        // 2. BUILD
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


        // 3. TEST
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


        // 4. BACKUP
        stage('Backup') {
            steps {
                script {

                    env.HAD_PREVIOUS = 'false'
                    env.BACKUP_FILE = ''

                    // Check whether an application already exists
                    int appExists = sh(
                        script: '''
                            ssh -i /home/jenkins-agent/.ssh/id_ed25519 \
                                -o BatchMode=yes \
                                deploy@192.168.31.190 \
                                'test -f /opt/myapp/index.html'
                        ''',
                        returnStatus: true
                    )

                    if (appExists == 0) {

                        env.HAD_PREVIOUS = 'true'

                        // Create unique timestamp
                        def timestamp = sh(
                            script: 'date +%Y%m%d-%H%M%S-%N',
                            returnStdout: true
                        ).trim()

                        // Build-specific backup filename
                        env.BACKUP_FILE =
                            "/opt/myapp-backups/index-build-${env.BUILD_NUMBER}-${timestamp}.html"

                        // Copy the old application on VM3
                        sh """
                            ssh -i /home/jenkins-agent/.ssh/id_ed25519 \
                                -o BatchMode=yes \
                                deploy@192.168.31.190 \
                                "set -eu; cp -a /opt/myapp/index.html '${env.BACKUP_FILE}'"
                        """

                        echo "Backup created: ${env.BACKUP_FILE}"

                    } else if (appExists == 1) {

                        echo 'No previous deployment exists. Backup skipped.'

                    } else {

                        error('Unable to connect to VM3 or inspect the application file.')

                    }
                }
            }
        }


        // 5. DEPLOY
        stage('Deploy') {
            steps {
                echo 'Deploying application to VM3'

                sh '''
                    set -eu

                    REMOTE_TMP="/opt/myapp/index.html.tmp-${BUILD_NUMBER}"

                    # Upload to a temporary filename
                    scp -i /home/jenkins-agent/.ssh/id_ed25519 \
                        -o BatchMode=yes \
                        build/index.html \
                        deploy@192.168.31.190:"$REMOTE_TMP"

                    # Atomically replace the live application
                    ssh -i /home/jenkins-agent/.ssh/id_ed25519 \
                        -o BatchMode=yes \
                        deploy@192.168.31.190 \
                        "mv '$REMOTE_TMP' /opt/myapp/index.html"

                    echo "Deployment completed"

                    ssh -i /home/jenkins-agent/.ssh/id_ed25519 \
                        -o BatchMode=yes \
                        deploy@192.168.31.190 \
                        'hostname && whoami && cat /opt/myapp/index.html'
                '''
            }
        }


        // 6. VERIFY AND AUTOMATIC ROLLBACK
        stage('Verify') {
            steps {
                script {

                    echo 'Checking application availability...'

                    // Verify HTTP 200 and exact application content
                    int verifyStatus = sh(
                        script: '''
                            set -eu

                            RESPONSE_FILE=$(mktemp)
                            trap 'rm -f "$RESPONSE_FILE"' EXIT

                            HTTP_CODE=$(curl -sS \
                                -o "$RESPONSE_FILE" \
                                -w "%{http_code}" \
                                http://192.168.31.190/)

                            echo "HTTP Status: $HTTP_CODE"

                            test "$HTTP_CODE" = "200"

                            cmp -s build/index.html "$RESPONSE_FILE"

                            echo "Application content verified successfully"
                        ''',
                        returnStatus: true
                    )


                    if (verifyStatus == 0) {

                        echo 'Deployment successful! No rollback required.'

                    } else {

                        echo 'Deployment verification FAILED!'
                        echo 'Starting automatic rollback...'


                        if (env.HAD_PREVIOUS == 'true') {

                            // Restore the precise backup created by this build
                            sh """
                                ssh -i /home/jenkins-agent/.ssh/id_ed25519 \
                                    -o BatchMode=yes \
                                    deploy@192.168.31.190 \
                                    "set -eu; \
                                    cp -a '${env.BACKUP_FILE}' \
                                    '/opt/myapp/index.html.rollback-${env.BUILD_NUMBER}'; \
                                    mv '/opt/myapp/index.html.rollback-${env.BUILD_NUMBER}' \
                                    /opt/myapp/index.html"
                            """

                            echo 'Previous application file restored.'

                            // Retrieve backup so we can compare HTTP response
                            sh """
                                scp -i /home/jenkins-agent/.ssh/id_ed25519 \
                                    -o BatchMode=yes \
                                    deploy@192.168.31.190:"${env.BACKUP_FILE}" \
                                    build/rollback-expected.html
                            """

                            // Verify restored application
                            int rollbackStatus = sh(
                                script: '''
                                    set -eu

                                    RESPONSE_FILE=$(mktemp)
                                    trap 'rm -f "$RESPONSE_FILE"' EXIT

                                    HTTP_CODE=$(curl -sS \
                                        -o "$RESPONSE_FILE" \
                                        -w "%{http_code}" \
                                        http://192.168.31.190/)

                                    echo "Rollback HTTP Status: $HTTP_CODE"

                                    test "$HTTP_CODE" = "200"

                                    cmp -s build/rollback-expected.html "$RESPONSE_FILE"

                                    echo "Previous version successfully restored and verified."
                                ''',
                                returnStatus: true
                            )

                            if (rollbackStatus != 0) {

                                error('Rollback file was restored, but post-rollback HTTP verification failed. Check Nginx and network connectivity.')

                            } else {

                                error('New deployment failed. Previous version restored successfully. Build marked FAILURE.')
                            }

                        } else {

                            // No previous application existed
                            sh """
                                ssh -i /home/jenkins-agent/.ssh/id_ed25519 \
                                    -o BatchMode=yes \
                                    deploy@192.168.31.190 \
                                    'rm -f /opt/myapp/index.html'
                            """

                            error('First deployment failed. No previous version existed; failed application file removed.')
                        }
                    }
                }
            }
        }
    }
}
