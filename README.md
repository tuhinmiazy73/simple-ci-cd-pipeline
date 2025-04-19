In this article, we’ll walk through creating a Jenkins CI/CD pipeline that automates cloning a GitHub repository into a remote production server via SSH, step-by-step.

📌 Step-by-Step Guide

✅ 1. Create a Git Repository

Start by pushing your project to a GitHub repository.

✅ 2. Add GitHub Credentials to Jenkins

Go to Manage Jenkins → Credentials and add your GitHub username & Personal Access Token (or password) as a credential. This enables Jenkins to interact with GitHub securely.

✅ 3. Setup SSH Access to the Remote Server

On your Jenkins server, generate an SSH key (if not already present):
# ssh-keygen -t rsa -b 4096

Copy the public key (id_rsa.pub) to your remote production server:
# ssh-copy-id root@192.168.150.131

Test it:
# ssh root@192.168.150.131

✅ 4. Add Jenkins SSH Private Key to Credential Manager
Add your SSH private key (id_rsa) to Jenkins credentials as SSH Username with private key.

✅ 5. Install Git on the Remote Server
Ensure Git is installed on your production server:
# yum install git 

✅ 6. Create the Jenkins Pipeline
Here’s a sample declarative pipeline:

pipeline {
    agent any

    environment {
        WORKER_NODE_IP = '192.168.150.131'
        GIT_REPO = 'https://github.com/yourusername/yourrepo.git'
    }

    stages {
        stage('Test SSH Login to Worker Node') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'JenkinsToWorkerSSH', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no root@${WORKER_NODE_IP} "echo '✅ SSH Login Successful'"
                    '''
                }
            }
        }

        stage('Clone Git Repo on Worker Node') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'JenkinsToWorkerSSH', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no root@${WORKER_NODE_IP} '
                            rm -rf test-repo && \
                            git clone ${GIT_REPO}
                        '
                    '''
                }
            }
        }
    }

    post {
        failure {
            echo "❌ Pipeline failed. Please check logs."
        }
        success {
            echo "✅ Pipeline executed successfully."
        }
    }
}
✅ Final Output
