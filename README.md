# jenkinsfile

pipeline {
    agent any

    environment {
        SSH_KEY_PATH = credentials('ec2_id') // configured ec2 .pem file content in Manage jenkins/credentials as SSH username with private key

        EC2_IP = '3.85.144.64' // Replace with your EC2 instance's public IP
        EC2_USER = 'ec2-user'
        APP_NAME = 'hello-1.0'
        WAR_FILE = "target/${APP_NAME}.war"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/aarifsh87/boxfuse-sample-java-war-hello.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Deploy to EC2') {
            steps {
                // Deploying the application to the EC2 instance using SCP and SSH
                script {
                    sh "scp -i ${SSH_KEY_PATH} target/hello-1.0.war ${EC2_USER}@${EC2_IP}:/tmp/"
                    sh "ssh -i ${SSH_KEY_PATH} ${EC2_USER}@${EC2_IP} 'sudo cp /tmp/hello-1.0.war /usr/share/nginx/html/ && sudo systemctl restart nginx'"
                }
            }
        }
    }

    post {
        always {
            cleanWs() // Clean up workspace
        }
    }
}
