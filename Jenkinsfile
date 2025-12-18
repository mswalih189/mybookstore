pipeline {
    agent any
    
    environment {
        BASTION_IP = '172.31.25.232'
        ANSIBLE_DIR = '/root/web/ansible'
        ALB_DNS = 'project-alb-1097922705.eu-north-1.elb.amazonaws.com'
        SSH_KEY = credentials('bastion-ssh-key')
    }
    
    stages {
        stage('Checkout Git') {
            steps {
                echo '✅ Git checkout complete - mybookstore master branch'
            }
        }
        
        stage('Deploy to Production') {
            steps {
                sh '''
                    echo "🚀 Deploying to bastion ${BASTION_IP}..."
                    ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i "${SSH_KEY}" ec2-user@${BASTION_IP} "
                        sudo su - root -c '
                            echo \"Cloning fresh repo...\"
                            cd ${ANSIBLE_DIR} &&
                            rm -rf /root/mybookstore &&
                            git clone https://github.com/mswalih189/mybookstore.git /root/mybookstore &&
                            cd /root/mybookstore && git checkout master &&
                            echo \"Running Ansible deployment...\"
                            cd ${ANSIBLE_DIR} &&
                            ansible-playbook -i inventory.ini deploy.yml &&
                            echo \"✅ Deployment complete!\"
                        '
                    "
                    echo '🎉 Bastion deployment successful!'
                '''
            }
        }
        
        stage('Health Check') {
            steps {
                script {
                    def response = sh(
                        script: "curl -s -o /dev/null -w '%{http_code}' http://${ALB_DNS}",
                        returnStdout: true
                    ).trim()
                    if (response != '200') {
                        error "❌ ALB Health Check FAILED: HTTP ${response}"
                    }
                    echo "✅ ALB Healthy: HTTP ${response}"
                }
            }
        }
    }
    
    post {
        success {
            echo "🎉🎉 BOOKSTORE DEPLOYMENT SUCCESSFUL!"
        }
        failure {
            echo "❌ Deployment FAILED"
        }
    }
}
