pipeline {
    agent any
    
    environment {
        BASTION_IP = '172.31.25.232'
        ANSIBLE_DIR = '/root/web/ansible'  // ✅ Updated path
        ALB_DNS = 'project-alb-1097922705.eu-north-1.elb.amazonaws.com'
    }
    
    stages {
        stage('Checkout Git') {
            steps {
                echo '✅ Git checkout complete'
            }
        }
        
        stage('Deploy to Production') {
            steps {
                sshagent(['bastion-ssh-key']) {
                    sh '''
                        ssh root@${BASTION_IP} "
                            cd ${ANSIBLE_DIR} && \\
                            rm -rf /root/mybookstore && \\
                            git clone https://github.com/mswalih189/mybookstore.git /root/mybookstore && \\
                            cd /root/mybookstore && git checkout master && \\
                            cd ${ANSIBLE_DIR} && \\
                            ansible-playbook -i inventory.ini deploy.yml && \\
                            curl -s http://${ALB_DNS} | head -10
                        "
                    '''
                }
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
                        error "ALB Health Check FAILED: HTTP ${response}"
                    }
                    echo "✅ ALB Healthy: HTTP ${response}"
                }
            }
        }
    }
    
    post {
        success {
            echo "🎉 Bookstore deployed successfully!"
        }
        failure {
            echo "❌ Deployment failed - check logs"
        }
    }
}
