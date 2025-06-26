pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Abhigyan-Chakraborty/Jenkins-Pipelined-Master-Java.git', branch: 'main'
            }
        }

        stage('Build WAR') {
            steps {
                bat 'mvn clean package -DskipTests=false'
            }
        }

        stage('Run Tests') {
            steps {
                bat 'mvn test'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def warFile = "target/my-webapp.war"
                    def tomcatUrl = 'http://localhost:8082/manager/text/deploy'
                    def contextPath = '/myapp' // You can use '' for ROOT

                    withCredentials([usernamePassword(credentialsId: 'tomcat-deployer-creds', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                        bat """
                            curl -v -u %TOMCAT_USER%:%TOMCAT_PASS% ^
                            --upload-file "${warFile}" ^
                            "${tomcatUrl}?path=${contextPath}&update=true"
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build, test, and deployment completed successfully.'
        }
        failure {
            echo '❌ Something went wrong.'
        }
    }
}
