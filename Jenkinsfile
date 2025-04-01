pipeline {
    agent any
    
    tools {
        maven "Mav363"  // Ensure this matches the tool name in Jenkins Global Tool Configuration
    }

    environment {
        SONARQUBE_SERVER = 'SONARQUBE-SERVER'  // Replace with your SonarQube server name
        SSH_CREDENTIAL_ID = 'winscp-cred'      // SSH credentials ID in Jenkins
        REMOTE_HOST = '172.31.46.113'
        SSH_PORT = '2222'                      // Custom SSH port
    }

    stages {
        stage('Clone Repo') {
            steps {
                git 'https://github.com/03amittrivedi/maven-web-app.git'
            }
        }
        stage('MVN Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Code Review') {
            steps {
                withSonarQubeEnv(SONARQUBE_SERVER) {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        stage('Deploy App') {
            steps {
                // Remove old SSH key if exists
                //sh "ssh-keygen -f '/var/lib/jenkins/.ssh/known_hosts' -R '[${REMOTE_HOST}]'"

                // Use SSH agent for secure connection
                sshagent (credentials: [SSH_CREDENTIAL_ID]) {
                    // Copy the WAR file to the remote server
                    sh "scp -P ${SSH_PORT} -o StrictHostKeyChecking=no target/maven-web-app.war root@${REMOTE_HOST}:/usr/local/tomcat/webapps"

                    // Restart the Tomcat service (uncomment if needed)
                    // sh "ssh -p ${SSH_PORT} -o StrictHostKeyChecking=no root@${REMOTE_HOST} 'systemctl restart tomcat9'"
                }
            }
        }
    }
    
    post {
        success {
            echo 'Build, SonarQube analysis, and deployment completed successfully!'
        }
        failure {
            echo 'There was an issue with the build, SonarQube analysis, or deployment.'
        }
    }
}
