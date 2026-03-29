pipeline {
    agent any

    environment {
        BLUE_IP = "YOUR_BLUE_SERVER_IP"
        GREEN_IP = "YOUR_GREEN_SERVER_IP"

        BLUE_TG = "arn:aws:elasticloadbalancing:eu-north-1:XXXX:targetgroup/Blue-TG/XXXX"
        GREEN_TG = "arn:aws:elasticloadbalancing:eu-north-1:XXXX:targetgroup/Green-TG/XXXX"

        LISTENER_ARN = "arn:aws:elasticloadbalancing:eu-north-1:XXXX:listener/app/XXXX"
        REGION = "eu-north-1"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git 'https://github.com/tushdarek/green-blue-deployment.git'
            }
        }

        stage('Determine Active Environment') {
            steps {
                script {
                    def currentTG = sh(
                        script: """
                        aws elbv2 describe-listeners \
                        --listener-arn ${LISTENER_ARN} \
                        --region ${REGION} \
                        --query "Listeners[0].DefaultActions[0].TargetGroupArn" \
                        --output text
                        """,
                        returnStdout: true
                    ).trim()

                    if (currentTG == BLUE_TG) {
                        env.INACTIVE_IP = GREEN_IP
                        env.NEW_TG = GREEN_TG
                        env.OLD_TG = BLUE_TG
                        echo "Active: BLUE → Deploying to GREEN"
                    } else {
                        env.INACTIVE_IP = BLUE_IP
                        env.NEW_TG = BLUE_TG
                        env.OLD_TG = GREEN_TG
                        echo "Active: GREEN → Deploying to BLUE"
                    }
                }
            }
        }

        stage('Deploy to Inactive') {
            steps {
                script {
                    sshagent(['blue-green-key']) {
                        sh """
                        scp -o StrictHostKeyChecking=no index.html ubuntu@${INACTIVE_IP}:/var/www/html/
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
                        script: "curl -s http://${INACTIVE_IP}",
                        returnStdout: true
                    )

                    if (!status.contains("version") && !status.contains("blue-green")) {
                        error "Health check failed!"
                    }

                    echo "Health check passed"
                }
            }
        }

        stage('Switch Traffic') {
            steps {
                script {
                    withAWS(credentials: 'aws-creds', region: REGION) {
                        sh """
                        aws elbv2 modify-listener \
                        --listener-arn ${LISTENER_ARN} \
                        --default-actions Type=forward,TargetGroupArn=${NEW_TG}
                        """
                    }
                }
            }
        }
    }

    post {
        failure {
            script {
                echo "Deployment failed! Rolling back..."

                withAWS(credentials: 'aws-creds', region: REGION) {
                    sh """
                    aws elbv2 modify-listener \
                    --listener-arn ${LISTENER_ARN} \
                    --default-actions Type=forward,TargetGroupArn=${OLD_TG}
                    """
                }
            }
        }

        success {
            echo "Deployment successful 🎉"
        }
    }
}