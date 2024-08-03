pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Checking out the repository
                checkout scm
            }
        }
        stage('Build') {
            agent { label 'build' }
            steps {
                // Building the project using Maven
                sh 'mvn clean package'
                sh 'ls -l target' // Listing contents of the target directory for verification
            }
        }
        stage('Test') {
            agent { label 'build' }
            steps {
                // Running tests using Maven
                sh 'mvn test'
            }
        }
        stage('Upload to Nexus') {
            agent { label 'deploy' }
            steps {
                script {
                    def warFile = 'target/WebAppCal-0.0.6.war'
                    // Checking if the WAR file exists before attempting to upload
                    if (fileExists(warFile)) {
                        nexusArtifactUploader artifacts: [
                            [artifactId: 'WebAppCal',
                             classifier: '',
                             file: warFile,
                             type: 'war']
                        ],
                        credentialsId: 'your-credentials-id',
                        groupId: 'com.web.cal',
                        nexusUrl: 'http://54.152.176.206:8081/nexus',
                        nexusVersion: 'nexus2',
                        protocol: 'http',
                        repository: 'your-repo',
                        version: '0.0.6'
                    } else {
                        // Error if the WAR file is not found
                        error "WAR file not found: ${warFile}"
                    }
                }
            }
        }
        stage('Deploy to Tomcat') {
            agent { label 'deploy' }
            steps {
                // Placeholder for deployment steps
                // Add your deployment commands here
                echo "Deploying to Tomcat"
            }
        }
    }

    post {
        always {
            // Sending an email notification after the pipeline execution
            emailext to: 'timothytoweh1@gmail.com',
                      subject: "Jenkins Build ${currentBuild.fullDisplayName}",
                      body: "Check console output at ${env.BUILD_URL} to view the results."
        }
    }
}
