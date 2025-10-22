pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        DEPLOY_USER = 'ec2-user'
        DEPLOY_HOST = '54.146.236.246'
        DEPLOY_PATH = '/home/ec2-user/demowebapp'
        JAR_NAME = 'demowebapp-0.0.1-SNAPSHOT.jar'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/aashismhj/demowebapp.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['ec2-key']) {
                    sh '''
                    scp -o StrictHostKeyChecking=no target/${JAR_NAME} ${DEPLOY_USER}@${DEPLOY_HOST}:${DEPLOY_PATH}/
                    ssh ${DEPLOY_USER}@${DEPLOY_HOST} '
                        pkill -f ${JAR_NAME} || true
                        nohup java -jar ${DEPLOY_PATH}/${JAR_NAME} > app.log 2>&1 &
                    '
                    '''
                }
            }
        }
    }
}
