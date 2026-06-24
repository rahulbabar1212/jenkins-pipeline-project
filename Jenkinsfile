pipeline {
    agent any

    environment {
        DEPLOY_SERVER = "ec2-user@<EC2_2_IP>"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/YOUR_USERNAME/jenkins-ec2-project.git'
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
