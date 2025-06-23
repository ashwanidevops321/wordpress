pipeline {
    agent any

    environment {
    DEPLOY_HOST_ID = 'deploy_host'
    DEPLOY_DIR_ID  = 'deploy_directory'
    FTP_CRED_ID    = 'ftp_creds'
}

    stages {
        stage('Checkout WordPress Code') {
            steps {
                git branch: 'main', url: 'https://github.com/ashwanidevops321/wordpress.git'
            }
        }

        stage('Clean Development Files') {
            steps {
                echo '🧹 Cleaning unnecessary dev files before FTP upload...'
                sh '''
                    rm -rf .git .github .gitignore .env node_modules README.md package*.json
                '''
            }
        }

        stage('Deploy to cPanel via FTP') {
            steps {
                echo "📤 Uploading WordPress files to ${env.DEPLOY_HOST}..."
                sh """
                    cat <<EOF > /tmp/ftp_script.lftp
set ftp:ssl-allow no
set ftp:passive-mode true
open -u ${FTP_USER},${FTP_PASS} ${DEPLOY_HOST}
mirror --reverse --delete --verbose \\
    --exclude-glob .git* \\
    --exclude-glob node_modules/ \\
    --exclude-glob .env \\
    --exclude-glob README.md \\
    --exclude-glob package*.json \\
    --exclude-glob .github/ \\
    ./ ${DEPLOY_DIR}
bye
EOF

lftp -f /tmp/ftp_script.lftp
"""
            }
        }
    }

    post {
        success {
            echo "✅ WordPress deployment to ${env.DEPLOY_HOST} completed!"
        }
        failure {
            echo "❌ Deployment to ${env.DEPLOY_HOST} failed!"
        }
        always {
            cleanWs()
        }
    }
}
