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
                    // Verify the directory structure
                    sh 'ls -l /home/ubuntu/workspace/ci-cd' // List contents of the parent directory
                    sh 'ls -l /home/ubuntu/workspace/ci-cd/Jenkins-CI-CD-Project' // List contents of the Jenkins-CI-CD-Project directory
                    
                    dir('/home/ubuntu/workspace/ci-cd') {
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
                    dir('/home/ubuntu/workspace/ci-cd') {
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
                    dir('/home/ubuntu/workspace/ci-cd') {
                        def version = readMavenPom().getVersion()
                        def warFile = "target/WebAppCal-${version}.war"
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
                            nexusUrl: '35.172.213.71:8081/nexus',
                            nexusVersion: 'nexus2',
                            protocol: 'http',
                            repository: 'releases',
                            version: version
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
                    def version = readMavenPom().getVersion()
                    def warFile = "WebAppCal-${version}.war"
                    def nexusUrl = "http://35.172.213.71:8081/nexus/service/local/repositories/releases/content/com/web/cal/WebAppCal/${version}/"
                    def downloadPath = "/tmp/${warFile}"
                    def tomcatWebappsDir = '~/apache-tomcat*/webapps/'
                    def tomcatBinDir = '~/apache-tomcat*/bin/'

                    // Download WAR file from Nexus
                    sh """
                        wget ${nexusUrl}${warFile} -O ${downloadPath}
                    """

                    // Deploy the WAR file to Tomcat
                    sh """
                        sudo rm -rf ${tomcatWebappsDir}*.war
                        sudo mv ${downloadPath} ${tomcatWebappsDir}
                        sudo systemctl daemon-reload
                        sudo ${tomcatBinDir}shutdown.sh
                        sudo ${tomcatBinDir}startup.sh
                    """
                }
            }
        }
    }
    post {
        success {
            mail to: 'timothytoweh1@gmail.com',
                 subject: 'Jenkins CI-CD Successful',
                 body: 'Jenkins CI-CD Job was successful, YAAAY'
        }
        failure {
            mail to: 'timothytoweh1@gmail.com',
                 subject: 'Jenkins CI-CD Failed',
                 body: 'Jenkins CI-CD job failed, sorry'
        }
    }
}
