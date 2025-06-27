// Jenkinsfile for Node.js application deployment to EC2

pipeline {
    agent any // Jenkins will use its own server (master) to run this pipeline

    environment {
        // Define your target EC2 instance's public IP/DNS here
        TARGET_EC2_IP = '13.221.144.198' // Replace this!
        APP_DIR = '/var/www/my-node-app'
        SSH_CREDENTIALS_ID = 'target-ec2-ssh-key' // The ID you set in Jenkins Credentials
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                echo "Checking out source code from Git..."
                checkout scm // This checks out your project from the configured SCM
            }
        }

        stage('Build and Package') {
            steps {
                echo "Installing Node.js dependencies..."
                sh 'npm install' // Installs dependencies based on package.json
                echo "Building application (if any build step was needed)..."
                // For a simple Node.js app, 'npm install' is usually enough.
                // If you had a React/Angular frontend, you'd run 'npm run build' here.
            }
        }

        stage('Run Tests') {
            steps {
                echo "Running application tests..."
                sh 'npm test' // Executes your test script defined in package.json
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo "Deploying application to ${TARGET_EC2_IP}..."
                // Use sshagent to securely pass the SSH key
                sshagent(credentials: [env.SSH_CREDENTIALS_ID]) {
                    // Clean up existing app files on the target EC2
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@${env.TARGET_EC2_IP} 'sudo rm -rf ${env.APP_DIR}/*'"
                    // Copy new app files to the target EC2
                    sh "scp -o StrictHostKeyChecking=no -r ./* ubuntu@${env.TARGET_EC2_IP}:${env.APP_DIR}/"
                    // Stop any running PM2 process for the app
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@${env.TARGET_EC2_IP} 'pm2 stop my-node-app || true'"
                    // Start the application using PM2
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@${env.TARGET_EC2_IP} 'cd ${env.APP_DIR} && pm2 start app.js --name my-node-app || pm2 restart my-node-app'"
                    // Save PM2 process list so it restarts on reboot
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@${env.TARGET_EC2_IP} 'pm2 save'"
                    echo "Deployment complete! Application should be accessible at http://${TARGET_EC2_IP}:3000"
                }
            }
        }
    }

    post {
        always {
            cleanWs() // Clean up the Jenkins workspace to save disk space
            echo 'Pipeline finished.'
        }
        success {
            echo 'Pipeline succeeded!'
            // You could add Slack/email notifications here
        }
        failure {
            echo 'Pipeline failed. Check the console output for errors.'
            // Add failure notifications
        }
    }
}