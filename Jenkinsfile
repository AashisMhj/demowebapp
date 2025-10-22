pipeline {
    agent any

    tools {
        maven 'Maven' // 👈 use the exact name from Jenkins → Manage Jenkins → Global Tool Configuration
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
                git branch: 'main', url: 'https://github.com/AashisMhj/demowebapp.git'
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
                    sh """
                        # Copy the JAR to EC2
                        scp -o StrictHostKeyChecking=no target/${JAR_NAME} ${DEPLOY_USER}@${DEPLOY_HOST}:${DEPLOY_PATH}/

                        # Restart the Spring Boot app remotely
                        ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_HOST} << 'EOF'
                            set -e
                            cd ${DEPLOY_PATH}

                            # Kill old process if running
                            PID=\$(pgrep -f ${JAR_NAME}) || true
                            if [ -n "\$PID" ]; then
                                echo "Stopping existing process \$PID..."
                                kill -9 \$PID || true
                            fi

                            # Start the new app in background
                            echo "Starting ${JAR_NAME}..."
                            nohup java -jar ${JAR_NAME} > app.log 2>&1 &
                            echo "App started!"
                        EOF
                    """
                }
            }
        }
    }
}
