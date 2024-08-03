pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package'
                sh 'ls -l target' // List contents of the target directory
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Upload to Nexus') {
            steps {
                script {
                    def warFile = 'target/WebAppCal-0.0.6.war'
                    if (fileExists(warFile)) {
                        nexusArtifactUploader artifacts: [
                            [artifactId: 'WebAppCal',
                             classifier: '',
                             file: warFile,
                             type: 'war']
                        ],
                        credentialsId: 'your-credentials-id',
                        groupId: 'com.web.cal',
                        nexusUrl: 'http://your-nexus-repo-url',
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        repository: 'your-repo',
                        version: '0.0.6'
                    } else {
                        error "WAR file not found: ${warFile}"
                    }
                }
            }
        }
        stage('Deploy to Tomcat') {
            steps {
                // Deployment steps
            }
        }
    }

    post {
        always {
            emailext to: 'timothytoweh1@gmail.com',
                      subject: "Jenkins Build ${currentBuild.fullDisplayName}",
                      body: "Check console output at ${env.BUILD_URL} to view the results."
        }
    }
}
