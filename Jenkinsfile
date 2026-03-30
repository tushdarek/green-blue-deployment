pipeline {
    agent any

    environment {
        REGION = 'eu-north-1'
        LISTENER_ARN = 'arn:aws:elasticloadbalancing:eu-north-1:098688552647:listener/app/blue-green/38707e768b5dfd29/985bab3a196613ae'

        BLUE_TG = 'arn:aws:elasticloadbalancing:eu-north-1:098688552647:targetgroup/Blue-tg/9a66b16af68a0c18'   // keep your original
        GREEN_TG = 'arn:aws:elasticloadbalancing:eu-north-1:098688552647:targetgroup/Green-TG/6c4c93c5e6d8a8f8'

        BLUE_IP = '16.171.133.4'
        GREEN_IP = '13.60.207.99 ' // keep your original
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/tushdarek/green-blue-deployment.git'
            }
        }

        stage('Determine Active Environment') {
            steps {
                script {
                    def activeTG = sh(
                        script: """
                        aws elbv2 describe-listeners \
                        --listener-arn ${LISTENER_ARN} \
                        --region ${REGION} \
                        --query Listeners[0].DefaultActions[0].TargetGroupArn \
                        --output text
                        """,
                        returnStdout: true
                    ).trim()

                    if (activeTG == BLUE_TG) {
                        env.ACTIVE = "BLUE"
                        env.INACTIVE = "GREEN"
                        env.TARGET_IP = GREEN_IP
                        env.NEW_TG = GREEN_TG
                        echo "Active: BLUE → Deploying to GREEN"
                    } else {
                        env.ACTIVE = "GREEN"
                        env.INACTIVE = "BLUE"
                        env.TARGET_IP = BLUE_IP
                        env.NEW_TG = BLUE_TG
                        echo "Active: GREEN → Deploying to BLUE"
                    }
                }
            }
        }

        stage('Deploy to Inactive') {
            steps {
                script {
                    sshagent(['ubuntu']) {
                        sh """
                        # Copy file to tmp first
                        scp -o StrictHostKeyChecking=no index.html ubuntu@${TARGET_IP}:/tmp/

                        # Move with sudo (fix permission issue)
                        ssh -o StrictHostKeyChecking=no ubuntu@${TARGET_IP} '
                            sudo mv /tmp/index.html /var/www/html/index.html
                        '
                        """
                    }
                }
            }
        }

        stage('Health Check') {
            steps {
                script {
                    sleep 10

                    def status = sh(
                        script: "curl -o /dev/null -s -w '%{http_code}' http://${TARGET_IP}",
                        returnStdout: true
                    ).trim()

                    if (status != "200") {
                        error("Health check failed!")
                    }

                    echo "Health check passed!"
                }
            }
        }

        stage('Switch Traffic') {
            steps {
                script {
                    sh """
                    aws elbv2 modify-listener \
                    --listener-arn ${LISTENER_ARN} \
                    --default-actions Type=forward,TargetGroupArn=${NEW_TG} \
                    --region ${REGION}
                    """
                }
            }
        }
    }

    post {
        failure {
            echo "Deployment failed! Rolling back..."
        }
        success {
            echo "Deployment successful!"
        }
    }
}