pipeline {
    agent any
    tools {
        jdk 'JAVA_11'
        maven 'Maven_3.9.8'
    }

    stages {
        stage('Cleanup workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from SCM') {
            steps {
                git branch: 'main', credentialsId: 'github_token', url: 'https://github.com/AryanKatariya/java-code.git'
            }
        }
        stage('Build application') {
            steps {
                sh "mvn clean package"
            }
        }
        stage('Test application') {
            steps {
                sh "mvn test"
            }
        }
        stage('Transfer WAR file') {
            steps {
                script {
                    def username = 'dockeradmin'
                    def password = 'your_password_here'

                    sh 'sudo yum update -y'
                    sh 'sudo yum install -y sshpass'
                    
                    def warFile = 'webapp/target/*.war'
                    sh "sshpass -p ${password} scp -o StrictHostKeyChecking=no ${warFile} ${username}@172.31.44.98:/home/dockeradmin"
                    }
                }
        }
    }    
}

