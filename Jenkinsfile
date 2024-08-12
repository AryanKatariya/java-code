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
                    withCredentials([usernamePassword(credentialsId: docker_ssh, passwordVariable: 'SERVER_PASS', usernameVariable: 'SERVER_USER')]) {
                        sh 'apt-get update && apt-get install -y sshpass'
                        def warFile = 'webapp/target/*.war'
                        sh "sshpass -p ${SERVER_PASS} scp -o StrictHostKeyChecking=no ${warFile} ${SERVER_USER}@172.31.44.98:/home/dockeradmin"
                        //sh "scp -o StrictHostKeyChecking=no ${warFile} dockeradmin@172.31.44.98:/home/dockeradmin"
                    }
                }
            }
        }    
    }
}
