
pipeline {
    agent any

    triggers {
    pollSCM('H/2 * * * *')
}

    environment {
        EC2_HOST = '34.224.156.171'
        EC2_USER = 'ubuntu'
        EC2_CREDENTIALS = 'brew-bloom-ec2-key'
        PROJECT_DIR = '/home/ubuntu/brew-and-bloom'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Deploy to AWS EC2') {
            steps {
                sshagent(credentials: [env.EC2_CREDENTIALS]) {
                    sh """
                        ssh -o StrictHostKeyChecking=accept-new ${EC2_USER}@${EC2_HOST} '
                            set -e
                            cd ${PROJECT_DIR}
                            git pull origin main
                            docker build -t brew-and-bloom:latest .
                            docker rm -f brew-and-bloom || true
                            docker run -d \
                                --name brew-and-bloom \
                                --restart unless-stopped \
                                -p 80:80 \
                                brew-and-bloom:latest
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Brew & Bloom successfully deployed to AWS EC2!'
        }
        failure {
            echo 'Deployment failed. Check the Jenkins console output.'
        }
    }
}
