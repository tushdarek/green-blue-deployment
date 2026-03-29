pipeline {
    agent any
    environment {
        // Target Groups
        BLUE_TG = "arn:aws:elasticloadbalancing:eu-north-1:098688552647:targetgroup/Blue-tg/9a66b16af68a0c18"
        GREEN_TG = "arn:aws:elasticloadbalancing:eu-north-1:098688552647:targetgroup/Green-TG/6c4c93c5e6d8a8f8"

        // ALB Listener
        ALB_LISTENER = "arn:aws:elasticloadbalancing:eu-north-1:098688552647:listener/app/blue-green/38707e768b5dfd29/985bab3a196613ae"

        // EC2 Instances
        BLUE_INSTANCE = "16.171.133.4"
        GREEN_INSTANCE = "13.60.207.99"

        // Initial Active TG (Blue)
        ACTIVE_TG = "${BLUE_TG}"
    }

    stages {
        stage('Deploy to Inactive') {
            steps {
                script {
                    // Determine inactive environment
                    def inactiveTG = (ACTIVE_TG == BLUE_TG) ? GREEN_TG : BLUE_TG
                    def inactiveIP = (inactiveTG == BLUE_TG) ? BLUE_INSTANCE : GREEN_INSTANCE

                    echo "Deploying to inactive environment: ${inactiveTG} (${inactiveIP})"

                    // Deploy code
                    sh "scp -i ~/.ssh/bluekey.pem -o StrictHostKeyChecking=no index.html ubuntu@${inactiveIP}:/var/www/html/"

                    // Save inactive TG for traffic switch
                    env.INACTIVE_TG = inactiveTG
                    env.INACTIVE_IP = inactiveIP
                }
            }
        }

        stage('Health Check') {
            steps {
                script {
                    def status = sh(script: "curl -f http://${env.INACTIVE_IP}", returnStatus: true)
                    if (status != 0) {
                        error "Health check failed!"
                    }
                }
            }
        }

        stage('Switch Traffic') {
            steps {
                script {
                    sh """
                        aws elbv2 modify-listener --listener-arn $ALB_LISTENER \
                        --default-actions Type=forward,TargetGroupArn=$INACTIVE_TG
                    """
                    echo "Traffic switched to inactive environment"

                    // Update ACTIVE_TG after successful switch
                    env.ACTIVE_TG = env.INACTIVE_TG
                }
            }
        }
    }

    post {
        failure {
            script {
                echo "Rollback: Traffic revert to previous active TG ($ACTIVE_TG)"
                sh """
                    aws elbv2 modify-listener --listener-arn $ALB_LISTENER \
                    --default-actions Type=forward,TargetGroupArn=$ACTIVE_TG
                """
            }
        }
    }
}