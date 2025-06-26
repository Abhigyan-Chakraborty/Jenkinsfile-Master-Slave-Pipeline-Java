pipeline {
    agent none

    environment {
        // Define these globally if needed
        WAR_NAME = "my-webapp.war"
        CONTEXT_PATH = "/hello-world"
        TOMCAT_URL = "http://localhost:8082/manager/text/deploy"
    }

    stages {
        stage('Checkout on Master') {
            agent { label 'master' }
            steps {
                checkout scm
            }
        }

        stage('Build on Windows Slave') {
            agent { label 'Windows_Node' }
            steps {
                bat 'mvn clean package -DskipTests=false'
            }
        }

        stage('Test on Windows Slave') {
            agent { label 'Windows_Node' }
            steps {
                bat 'mvn test'
            }
        }

        stage('Deploy from Windows Slave') {
            agent { label 'Windows_Node' }
            steps {
                script {
                    def warPath = "target/${env.WAR_NAME}"

                    withCredentials([usernamePassword(
                        credentialsId: 'tomcat-deployer-creds',
                        usernameVariable: 'TOMCAT_USER',
                        passwordVariable: 'TOMCAT_PASS'
                    )]) {
                        bat """
                            curl -v -u %TOMCAT_USER%:%TOMCAT_PASS% ^
                            --upload-file "${warPath}" ^
                            "${env.TOMCAT_URL}?path=${env.CONTEXT_PATH}&update=true"
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build, Test, and Deployment completed successfully.'
        }
        failure {
            echo '❌ Build failed or deployment issue.'
        }
    }
}
