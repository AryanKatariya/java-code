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
                    def password = 'docker'
                    def CONTAINER_NAME = 'devops-container'
                    
                    def warFile = 'webapp/target/*.war'
                    sh "sshpass -p ${password} scp -o StrictHostKeyChecking=no ${warFile} ${username}@172.31.44.98:/home/dockeradmin"

                    sh """
                            sshpass -p ${password} ssh -o StrictHostKeyChecking=no ${username}@172.31.44.98"
                            docker pull tomcat:8.0
                            docker run --name ${CONTAINER_NAME} -d -p 8080:8080 tomcat:8.0
                            docker cp ./*.war ${CONTAINER_NAME}:/usr/local/tomcat/webapps/webapp.war
                            docker restart ${CONTAINER_NAME}
                            "
                        """
                    }
                }
        }
    }    
}

