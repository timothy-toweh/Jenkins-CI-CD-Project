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
            agent { label 'deploy' }
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
                echo 'Deploying the application'
                script {
                    dir('Jenkins-CI-CD-Project') {
                        unstash 'Jenkins-CI-CD-Project'
                        
                        // Assuming the WAR file name and location
                        def warFile = 'target/WebAppCal-0.0.6.war'
                        
                        // Deploy the WAR file to Tomcat
                        sh """
                            sudo rm -rf ~/apache-tomcat*/webapps/*.war
                            sudo mv ${warFile} ~/apache-tomcat*/webapps/
                            sudo systemctl daemon-reload
                            sudo ~/apache-tomcat*/bin/shutdown.sh
                            sudo ~/apache-tomcat*/bin/startup.sh
                        """
                    }
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
