pipeline {
    agent any

    parameters {
        string(name: 'DEPLOY_HOST', defaultValue: '3.110.11.141', description: 'FTP host')
        string(name: 'DEPLOY_DIR', defaultValue: 'project', description: 'Remote FTP directory')
    }

    environment {
        DEPLOY_HOST = "${params.DEPLOY_HOST}"
        DEPLOY_DIR  = "${params.DEPLOY_DIR}"
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
                withCredentials([usernamePassword(credentialsId: 'ftp-deploy-creds', usernameVariable: 'FTP_USER', passwordVariable: 'FTP_PASS')]) {
                    sh '''
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
                            ./ ${DEPLOY_DIR};
                        bye
                    " -u "$FTP_USER","$FTP_PASS" "$DEPLOY_HOST"
                    '''
                }
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
