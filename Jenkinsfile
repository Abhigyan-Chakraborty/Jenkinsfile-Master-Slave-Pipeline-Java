pipeline {
    agent none

    environment {
        WAR_NAME = "my-webapp.war"
        CONTEXT_PATH = "/hello-world"
        TOMCAT_URL = "http://localhost:8082/manager/text/deploy"
    }

    stages {
        stage('Build on Master') {
            agent { label 'master' }
            stages {
                stage('Checkout') {
                    steps {
                        checkout scm
                    }
                }
                stage('Build WAR') {
                    steps {
                        bat 'mvn clean package -DskipTests=true'
                    }
                }
                stage('Archive WAR') {
                    steps {
                        archiveArtifacts artifacts: "target/${env.WAR_NAME}", fingerprint: true
                    }
                }
            }
        }

        stage('Test and Deploy on Windows-Node') {
            agent { label 'Windows-Node' }
            stages {
                stage('Get WAR from Master') {
                    steps {
                        unstash 'archive'
                    }
                }

                stage('Test WAR') {
                    steps {
                        bat 'mvn test'
                    }
                }

                stage('Deploy to Tomcat') {
                    steps {
                        script {
                            withCredentials([usernamePassword(
                                credentialsId: 'tomcat-deployer-creds',
                                usernameVariable: 'TOMCAT_USER',
                                passwordVariable: 'TOMCAT_PASS'
                            )]) {
                                bat """
                                    curl -v -u %TOMCAT_USER%:%TOMCAT_PASS% ^
                                    --upload-file "target\\${env.WAR_NAME}" ^
                                    "${env.TOMCAT_URL}?path=${env.CONTEXT_PATH}&update=true"
                                """
                            }
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build, test, and deploy completed successfully!"
        }
        failure {
            echo "❌ Something went wrong during the pipeline."
        }
    }
}
