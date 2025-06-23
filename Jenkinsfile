pipeline {
    agent any

    environment {
        DEPLOY_HOST_ID = 'deploy_host'
        DEPLOY_DIR_ID  = 'deploy_directory'
        SFTP_CRED_ID   = 'sftp_user_pass'
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

        stage('Deploy via SFTP (Password Auth)') {
            steps {
                echo '📤 Uploading only index.php and assets/ via SFTP (password)...'

                withCredentials([
                    string(credentialsId: "${DEPLOY_HOST_ID}", variable: 'DEPLOY_HOST'),
                    string(credentialsId: "${DEPLOY_DIR_ID}", variable: 'DEPLOY_DIR'),
                    usernamePassword(credentialsId: "${SFTP_CRED_ID}", usernameVariable: 'SFTP_USER', passwordVariable: 'SFTP_PASS')
                ]) {
                    sh '''
                        echo "Uploading to $DEPLOY_HOST:$DEPLOY_DIR using SFTP..."

                        # Ensure sshpass is available
                        if ! command -v sshpass &> /dev/null; then
                            echo "❌ sshpass not installed. Please install it on the agent."
                            exit 1
                        fi

                        mkdir -p /tmp/sftp_batch

                        # Create batch SFTP commands
                        echo "mkdir -p $DEPLOY_DIR/assets" > /tmp/sftp_batch/upload.sftp
                        echo "put index.php $DEPLOY_DIR/index.php" >> /tmp/sftp_batch/upload.sftp

                        find assets -type f | while read file; do
                            remote_path="$DEPLOY_DIR/$file"
                            remote_dir=$(dirname "$remote_path")
                            echo "mkdir -p $remote_dir" >> /tmp/sftp_batch/upload.sftp
                            echo "put $file $remote_path" >> /tmp/sftp_batch/upload.sftp
                        done

                        sshpass -p "$SFTP_PASS" sftp -o StrictHostKeyChecking=no -b /tmp/sftp_batch/upload.sftp $SFTP_USER@$DEPLOY_HOST
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ index.php and assets/ uploaded successfully via SFTP!"
        }
        failure {
            echo "❌ SFTP deployment failed!"
        }
        always {
            cleanWs()
        }
    }
}