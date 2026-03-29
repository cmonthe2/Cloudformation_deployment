pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        STACK_NAME = 's3'
        TEMPLATE_FILE = 'template.yml'
    }

    stages {
        stage('Validate Template') {
            steps {
                sh "aws cloudformation validate-template --template-body file://${TEMPLATE_FILE} --region ${AWS_REGION}"
            }
        }

        stage('Check Stack Exists') {
            steps {
                script {
                    def stackExists = sh(
                        script: "aws cloudformation describe-stacks --stack-name ${STACK_NAME} --region ${AWS_REGION} 2>&1",
                        returnStatus: true
                    ) == 0
                    env.STACK_EXISTS = stackExists.toString()
                }
            }
        }

        stage('Deploy Stack') {
            steps {
                script {
                    if (env.STACK_EXISTS == 'true') {
                        sh """
                            aws cloudformation update-stack \
                                --stack-name ${STACK_NAME} \
                                --template-body file://${TEMPLATE_FILE} \
                                --region ${AWS_REGION}
                        """
                    } else {
                        sh """
                            aws cloudformation create-stack \
                                --stack-name ${STACK_NAME} \
                                --template-body file://${TEMPLATE_FILE} \
                                --region ${AWS_REGION}
                        """
                    }
                }
            }
        }

        stage('Wait for Completion') {
            steps {
                script {
                    def waiter = env.STACK_EXISTS == 'true' ? 'stack-update-complete' : 'stack-create-complete'
                    sh "aws cloudformation wait ${waiter} --stack-name ${STACK_NAME} --region ${AWS_REGION}"
                }
            }
        }
    }

    post {
        success {
            echo "Stack ${STACK_NAME} deployed successfully."
        }
        failure {
            echo "Deployment failed. Check CloudFormation events for details."
        }
    }
}
