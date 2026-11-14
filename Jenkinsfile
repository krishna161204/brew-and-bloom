
pipeline {
    agent any

    triggers {
        pollSCM('H/2 * * * *')
    }

    environment {
        EC2_HOST = '34.224.156.171'
        EC2_USER = 'ubuntu'
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
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'brew-bloom-ec2-key',
                        keyFileVariable: 'SSH_KEY'
                    )
                ]) {
                    sh '''
                        ssh -i "$SSH_KEY" \
                            -o StrictHostKeyChecking=accept-new \
                            "$EC2_USER@$EC2_HOST" '
                                set -e
                                cd /home/ubuntu/brew-and-bloom
                                git pull origin main
                                docker build -t brew-and-bloom:latest .
                                docker rm -f brew-and-bloom || true
                                docker run -d \
                                    --name brew-and-bloom \
                                    --restart unless-stopped \
                                    -p 80:80 \
                                    brew-and-bloom:latest
                            '
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Brew & Bloom successfully deployed to AWS EC2!'
        }
        failure {
            echo 'Deployment failed. Check the console output.'
        }
    }
}
