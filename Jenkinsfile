pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "54.152.176.206:8081/nexus"
        NEXUS_REPOSITORY = "maven-releases"
        NEXUS_CREDENTIALS_ID = "nexus-credentials" // Ensure this matches the ID from Jenkins credentials
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/timothy-toweh/Jenkins-CI-CD-Project'
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
                    nexusPublisher nexusInstanceId: 'nexus', nexusRepositoryId: env.NEXUS_REPOSITORY, packages: [
                        [$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: 'war', filePath: 'target/WebAppCal.war']], mavenCoordinate: [groupId: 'com.example', artifactId: 'WebAppCal', version: '1.0.0']]
                    ], credentialsId: env.NEXUS_CREDENTIALS_ID
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