pipeline {
    agent any

    parameters {
        string(name: 'DEPLOY_HOST', defaultValue: '', description: 'FTP host (e.g., ftp.example.com)')
        string(name: 'DEPLOY_DIR', defaultValue: '', description: 'Remote FTP directory (e.g., public_html/wp-site)')
        string(name: 'FTP_USER', defaultValue: '', description: 'FTP Username')
        password(name: 'FTP_PASS', defaultValue: '', description: 'FTP Password')
    }

    environment {
        DEPLOY_HOST = "${params.DEPLOY_HOST}"
        DEPLOY_DIR  = "${params.DEPLOY_DIR}"
        FTP_USER    = "${params.FTP_USER}"
        FTP_PASS    = "${params.FTP_PASS}"
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
        // lftp is used for FTP transfers
        stage('Deploy to cPanel via FTP') {
            steps {
                echo "📤 Uploading WordPress files to $DEPLOY_HOST..."

                sh """
                lftp -e "
                    set ftp:ssl-allow no;
                    set ftp:passive-mode true;
                    mirror --reverse --delete --verbose \
                        --exclude-glob .git* \
                        --exclude-glob node_modules/ \
                        --exclude-glob .env \
                        --exclude-glob README.md \
                        --exclude-glob package*.json \
                        --exclude-glob .github/ \
                        ./ $DEPLOY_DIR;
                    bye
                " -u "$FTP_USER","$FTP_PASS" "$DEPLOY_HOST"
                """
            }
        }
    }

    post {
        success {
            echo "✅ WordPress deployment to $DEPLOY_HOST completed!"
        }
        failure {
            echo "❌ Deployment failed! Please check the console output."
        }
        always {
            cleanWs()
        }
    }
}
