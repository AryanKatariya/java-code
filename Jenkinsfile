pipeline {
    agent any

    tools {
        jdk 'JAVA_11'
        maven 'Maven_3.9.8'
    }

    environment {
        GITHUB_TOKEN = credentials('github_token')
    }

    stages {
        stage('Cleanup workspace') {
            steps {
                cleanWs()
            }
        }

        stage ('DAST - OWASP ZAP') {
            steps {
                script {
                    sh """
                    sshpass -p 'zap' ssh -o StrictHostKeyChecking=no zap@172.31.12.108 'ls -a'
                    """
                }
            }
        }


        stage('Checkout from SCM') {
            steps {
                git branch: 'main', credentialsId: 'github_token', url: 'https://github.com/AryanKatariya/java-code.git'
            }
        }
        
        stage('Check secrets') {
            steps {
                sh 'trufflehog --json https://$GITHUB_TOKEN@github.com/AryanKatariya/java-code.git > trufflehog_output.json'
            }
        }

        stage ('SCA - OWASPDC') {
            steps {
                dependencyCheck additionalArguments: ''' 
                    -o "./" 
                    -s "./"
                    -f "ALL" 
                    --prettyPrint''', odcInstallation: 'OWASP-DC'

                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
        }

        stage ('SAST - SonarQube') {
            steps {
                withSonarQubeEnv('sonar') {
                sh 'mvn clean sonar:sonar -Dsonar.java.binaries=src'
                }
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
        stage('Deploy to server') {
            steps {
                script {
                    def username = 'dockeradmin'
                    def password = 'docker'
                    def CONTAINER_NAME = 'devops-container'
                    
                    def warFile = 'webapp/target/*.war'
                    sh "sshpass -p ${password} scp -o StrictHostKeyChecking=no ${warFile} ${username}@172.31.44.98:/home/dockeradmin"

                    sh """
                            sshpass -p 'docker' ssh -o StrictHostKeyChecking=no dockeradmin@172.31.44.98 '
                            docker stop devops-container
                            docker rm devops-container
                            docker pull tomcat:8.0 &&
                            docker run --name devops-container -d -p 8080:8080 tomcat:8.0 &&
                            docker cp ./*.war devops-container:/usr/local/tomcat/webapps/webapp.war &&
                            docker restart devops-container
                            '
                        """
                    }
                }
        }
    }    
}

