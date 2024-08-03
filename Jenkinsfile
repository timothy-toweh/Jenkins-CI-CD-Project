pipeline {
    agent any
    tools {
        maven 'maven'
    }
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "54.152.176.206:8081/nexus"
        NEXUS_REPOSITORY = "Releases"
        NEXUS_CREDENTIALS_ID = "nexus-credentials" // Ensure this matches the ID from Jenkins credentials
    }
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code'
                checkout scm
            }
        }
        stage('Build') {
            agent { label 'build' }
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            agent { label 'build' }
            steps {
                sh 'mvn test'
            }
        }
        stage('Upload to Nexus') {
            steps {
                script {
                    nexusArtifactUploader(
                        nexusVersion: env.NEXUS_VERSION,
                        protocol: env.NEXUS_PROTOCOL,
                        nexusUrl: env.NEXUS_URL,
                        groupId: 'com.example',
                        version: '1.0.0',
                        repository: env.NEXUS_REPOSITORY,
                        credentialsId: env.NEXUS_CREDENTIALS_ID,
                        artifacts: [
                            [artifactId: 'WebAppCal', classifier: '', file: 'target/WebAppCal.war', type: 'war']
                        ]
                    )
                }
            }
        }
        stage('Deploy to Tomcat') {
            agent { label 'deploy' }
            steps {
                sh '''
                scp -o StrictHostKeyChecking=no target/WebAppCal.war ubuntu@52.90.23.134:/home/ubuntu/apache-tomcat-7.0.94/webapps/
                '''
                // Optionally, restart Tomcat or use other deployment mechanisms
            }
        }
    }
    post {
        success {
            emailext (
                to: 'timothytoweh1@gmail.com',
                subject: "Jenkins Build Successful: ${env.JOB_NAME} ${env.BUILD_NUMBER}",
                body: "The Jenkins build for ${env.JOB_NAME} ${env.BUILD_NUMBER} was successful.\n\nCheck it out at ${env.BUILD_URL}."
            )
        }
        failure {
            emailext (
                to: 'timothytoweh1@gmail.com',
                subject: "Jenkins Build Failed: ${env.JOB_NAME} ${env.BUILD_NUMBER}",
                body: "The Jenkins build for ${env.JOB_NAME} ${env.BUILD_NUMBER} has failed.\n\nCheck it out at ${env.BUILD_URL}."
            )
        }
    }
}
