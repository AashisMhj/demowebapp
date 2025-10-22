pipeline {
    agent any

    environment {
        IMAGE_NAME = 'aashismhj/demowebapp'    // Change to your Docker Hub repo
        DEPLOY_USER = 'ec2-user'
        DEPLOY_HOST = '18.209.7.79'
        DEPLOY_PATH = '/home/ec2-user/demowebapp'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/AashisMhj/demowebapp.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:latest .'
            }
        }

        stage('Push Docker Image') {
            environment {
                DOCKERHUB_CREDENTIALS = credentials('docker-id') // Jenkins credentials ID
            }
            steps {
                sh '''
                echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                docker push ${IMAGE_NAME}:latest
                docker logout
                '''
            }
        }

        stage('Deploy on EC2') {
            steps {
                sshagent(['ec2-key']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_HOST} << 'EOF'
                        docker pull ${IMAGE_NAME}:latest
                        docker stop demowebapp || true
                        docker rm demowebapp || true
                        docker run -d --name demowebapp -p 8080:8080 ${IMAGE_NAME}:latest
                    EOF
                    '''
                }
            }
        }
    }
}
