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

        stage('Deploy via LFTP over SFTP') {
            steps {
                echo '📤 Uploading index.php and assets/ via lftp (SFTP)...'

                withCredentials([
                    string(credentialsId: "${DEPLOY_HOST_ID}", variable: 'DEPLOY_HOST'),
                    string(credentialsId: "${DEPLOY_DIR_ID}", variable: 'DEPLOY_DIR'),
                    usernamePassword(credentialsId: "${SFTP_CRED_ID}", usernameVariable: 'SFTP_USER', passwordVariable: 'SFTP_PASS')
                ]) {
                    sh '''
                        # Check if lftp is installed
                        if ! command -v lftp &> /dev/null; then
                            echo "❌ lftp is not installed on this agent."
                            exit 1
                        fi

                        echo "✅ lftp is available. Proceeding with deployment..."

                        lftp -u "$SFTP_USER","$SFTP_PASS" sftp://$DEPLOY_HOST <<EOF
set ssl:verify-certificate no
set sftp:auto-confirm yes
set net:max-retries 2
set net:timeout 20

mirror --reverse --delete --verbose \\
       --include index.php \\
       --include-glob assets/* \\
       --exclude-glob .git* \\
       --exclude-glob .env \\
       --exclude-glob node_modules/ \\
       --exclude-glob README.md \\
       --exclude-glob package*.json \\
       --exclude-glob .github/ \\
       ./ $DEPLOY_DIR

bye
EOF
                    '''
                }
            }
        }
    } // END of stages

    post {
        success {
            echo "✅ index.php and assets/ uploaded successfully via lftp (SFTP)!"
        }
        failure {
            echo "❌ lftp SFTP deployment failed!"
        }
        always {
            cleanWs()
        }
    }
} // END of pipeline