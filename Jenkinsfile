
pipeline {
    agent any

    environment {
        REMOTE_USER_HOST = 'azureuser@20.56.34.147'
        SSH_CREDENTIALS_ID = 'remote-vm-ssh-key'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
                echo "Code downloaded from repository."
            }
        }

        stage('Deploy Apache Server') {
            steps {
                script {
                    sshagent(credentials: [SSH_CREDENTIALS_ID]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ${REMOTE_USER_HOST} '''
                                sudo apt-get update -y
                                
                                sudo apt-get install -y apache2

                                sudo systemctl enable --now apache2
                                
                                sudo systemctl status apache2
                            '''
                        """
                    }
                    echo "Apache server installed."
                }
            }
        }

        stage('Check Logs for Errors') {
            steps {
                script {
                    sshagent(credentials: [SSH_CREDENTIALS_ID]) {
                        def error_logs = sh(
                            script: "ssh -o StrictHostKeyChecking=no ${REMOTE_USER_HOST} 'grep -E \" \\\"(4|5)[0-9]{2} \\\"\" /var/log/apache2/access.log || true'",
                            returnStdout: true
                        ).trim()

                        if (error_logs) {
                            echo "!!! ERROR 4xx/5xx IN APACHE LOGS !!!"
                            echo "${error_logs}"
                            error("Pipeline stopped due to 4xx/5xx errors in logs.")
                        } else {
                            echo "Log checkup complete. Errors 4xx/5xx not found."
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline work finished.'
        }
    }
}
