pipeline {
    agent any

    environment {
        EC2_HOST = "16.171.10.78"           // ✅ change to your EC2 IP
        EC2_USER = "ubuntu"
        SSH_KEY_ID = "ec2-ssh-key"
        PROJECT_DIR = "/home/ubuntu/Branchloanapp"   // folder on EC2
    }

    triggers {
        githubPush()        // ✅ GitHub webhook triggers build
    }

    stages {

        stage('Clone Repo') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/GandhamRakesh11/Branchloanapp.git'
            }
        }

        stage('Copy Files to EC2') {
            steps {
                sshagent([SSH_KEY_ID]) {
                    sh """
                    rsync -avz --delete -e "ssh -o StrictHostKeyChecking=no" \
                        ./ $EC2_USER@$EC2_HOST:$PROJECT_DIR
                    """
                }
            }
        }

        stage('Deploy on EC2') {
            steps {
                sshagent([SSH_KEY_ID]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no $EC2_USER@$EC2_HOST '
                        cd $PROJECT_DIR &&
                        docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d --build
                    '
                    """
                }
            }
        }

        stage('Health Check') {
            steps {
                sshagent([SSH_KEY_ID]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no $EC2_USER@$EC2_HOST '
                        curl -f http://localhost:8000/health
                    '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment Success!"
        }
        failure {
            echo "Deployment Failed!"
        }
    }
}
