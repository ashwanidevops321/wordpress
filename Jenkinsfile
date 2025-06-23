pipeline {
    agent any

    environment {
        DEPLOY_HOST_ID = 'deploy_host'
        DEPLOY_DIR_ID  = 'deploy_directory'
        SFTP_CRED_ID   = 'ftp_creds'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/ashwanidevops321/wordpress.git'
            }
        }

        stage('Clean Dev Files') {
            steps {
                echo '🧹 Cleaning unnecessary files...'
                sh '''
                    rm -rf .git .github .gitignore .env node_modules README.md package*.json
                '''
            }
        }

        stage('Deploy via SFTP') {
            steps {
                echo '📤 Uploading only index.php and assets/ via SFTP (password auth)...'

                withCredentials([
                    string(credentialsId: "${DEPLOY_HOST_ID}", variable: 'DEPLOY_HOST'),
                    string(credentialsId: "${DEPLOY_DIR_ID}", variable: 'DEPLOY_DIR'),
                    usernamePassword(credentialsId: "${SFTP_CRED_ID}", usernameVariable: 'SFTP_USER', passwordVariable: 'SFTP_PASS')
                ]) {
                    sh '''
                        if ! command -v sshpass &> /dev/null; then
                            echo "❌ sshpass not installed. Please install it on the agent."
                            exit 1
                        fi

                        mkdir -p /tmp/sftp_batch
                        BATCH_FILE=/tmp/sftp_batch/upload.sftp

                        # Prepare batch file
                        echo "mkdir -p $DEPLOY_DIR/assets" > $BATCH_FILE
                        echo "put index.php $DEPLOY_DIR/index.php" >> $BATCH_FILE

                        find assets -type f | while read file; do
                            remote="$DEPLOY_DIR/$file"
                            echo "mkdir -p \$(dirname \"$remote\")" >> $BATCH_FILE
                            echo "put $file $remote" >> $BATCH_FILE
                        done

                        sshpass -p "$SFTP_PASS" \
                          sftp -o StrictHostKeyChecking=no -b $BATCH_FILE \
                          "$SFTP_USER@$DEPLOY_HOST"
                    '''
                }
            }
        }
    }

    post {
        success { echo "✅ index.php and assets/ uploaded successfully via SFTP!" }
        failure { echo "❌ SFTP deployment failed!" }
        always { cleanWs() }
    }
}