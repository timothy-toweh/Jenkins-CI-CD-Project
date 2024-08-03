pipeline {
    agent none

    stages {
        stage('Checkout') {
            agent { label 'build' }
            steps {
                // Checking out the repository directly on the build node
                checkout scm
            }
        }
        stage('Build') {
            agent { label 'build' }
            steps {
                script {
                    dir('Jenkins-CI-CD-Project') {
                        // Building the project using Maven
                        sh 'mvn clean package -Dmaven.compiler.source=1.8 -Dmaven.compiler.target=1.8'
                        sh 'ls -l target' // Listing contents of the target directory for verification
                    }
                }
            }
        }
        stage('Test') {
            agent { label 'build' }
            steps {
                script {
                    dir('Jenkins-CI-CD-Project') {
                        // Running tests using Maven
                        sh 'mvn test'
                    }
                }
            }
        }
        stage('Upload to Nexus') {
            agent { label 'build' }
            steps {
                script {
                    dir('Jenkins-CI-CD-Project') {
                        def warFile = 'target/WebAppCal-0.0.6.war'
                        // Checking if the WAR file exists before attempting to upload
                        if (fileExists(warFile)) {
                            nexusArtifactUploader artifacts: [
                                [artifactId: 'WebAppCal',
                                 classifier: '',
                                 file: warFile,
                                 type: 'war']
                            ],
                            credentialsId: 'nexus-credentials',
                            groupId: 'com.web.cal',
                            nexusUrl: 'http://54.152.176.206:8081/nexus',
                            nexusVersion: 'nexus2',
                            protocol: 'http',
                            repository: 'Releases',
                            version: '0.0.6'
                        } else {
                            // Error if the WAR file is not found
                            error "WAR file not found: ${warFile}"
                        }
                    }
                }
            }
        }
        stage('Deploy') {
            agent { label 'deploy' }
            steps {
                script {
                    // Define deployment steps on the deploy node
                    def warFile = 'WebAppCal-0.0.6.war'
                    sh """
                        # Download WAR file from Nexus
                        wget http://54.152.176.206:8081/nexus/repository/Releases/com/web/cal/WebAppCal/0.0.6/${warFile} -O /tmp/${warFile}
                        
                        # Deploy the WAR file to Tomcat
                        sudo rm -rf ~/apache-tomcat*/webapps/*.war
                        sudo mv /tmp/${warFile} ~/apache-tomcat*/webapps/
                        sudo systemctl daemon-reload
                        sudo ~/apache-tomcat*/bin/shutdown.sh
                        sudo ~/apache-tomcat*/bin/startup.sh
                    """
                }
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
