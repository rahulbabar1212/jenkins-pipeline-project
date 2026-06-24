pipeline {
    agent any

    environment {
        DEPLOY_SERVER = "ec2-user@172.31.23.4"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Build step..."
            }
        }

        stage('Test') {
            steps {
                echo "Test step..."
            }
        }

stage('Deploy') {
            steps {
                sshagent(credentials: ['ec2-target-server-key']) {
                    sh """
                    scp -o StrictHostKeyChecking=no index.html ${DEPLOY_SERVER}:/home/ec2-user/

                    ssh -o StrictHostKeyChecking=no ${DEPLOY_SERVER} '
                        sudo cp /home/ec2-user/index.html /var/www/html/index.html
                        sudo systemctl restart httpd
                    '
                    """
                }
            }
        }
}
