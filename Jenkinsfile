pipeline {
    agent any

    parameters {
        choice(
            name: 'ACTION',
            choices: ['plan', 'apply', 'destroy'],
            description: 'Terraform action to perform'
        )
        booleanParam(
            name: 'AUTO_APPROVE',
            defaultValue: false,
            description: 'Auto approve apply/destroy'
        )
    }

    environment {
        TF_VERSION = '1.6.0'
        AWS_REGION = 'us-east-1'
        PROJECT_NAME = 'test-3'

        TF_IN_AUTOMATION = 'true'
        TF_INPUT = 'false'

        // Jenkins credentials (MUST exist in Jenkins)
        AWS_CREDENTIALS     = credentials('aws-credentials')
        TF_STATE_BUCKET     = credentials('tf-state-bucket')
        TF_STATE_LOCK_TABLE = credentials('tf-state-lock-table')
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '30'))
        timeout(time: 60, unit: 'MINUTES')
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                dir('test-3-terraform') {
                    sh '''
                        terraform init \
                          -backend-config="bucket=${TF_STATE_BUCKET}" \
                          -backend-config="dynamodb_table=${TF_STATE_LOCK_TABLE}" \
                          -backend-config="region=${AWS_REGION}"
                    '''
                }
            }
        }

        stage('Terraform Plan') {
            when {
                expression { params.ACTION == 'plan' || params.ACTION == 'apply' }
            }
            steps {
                dir('test-3-terraform') {
                    sh 'terraform plan'
                }
            }
        }

        stage('Terraform Apply') {
            when {
                expression { params.ACTION == 'apply' }
            }
            steps {
                dir('test-3-terraform') {
                    script {
                        if (params.AUTO_APPROVE) {
                            sh 'terraform apply -auto-approve'
                        } else {
                            sh 'terraform apply'
                        }
                    }
                }
            }
        }

        stage('Terraform Destroy') {
            when {
                expression { params.ACTION == 'destroy' }
            }
            steps {
                dir('test-3-terraform') {
                    script {
                        if (params.AUTO_APPROVE) {
                            sh 'terraform destroy -auto-approve'
                        } else {
                            sh 'terraform destroy'
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Terraform ${params.ACTION} completed successfully"
        }
        failure {
            echo "Terraform ${params.ACTION} failed"
        }
        always {
            deleteDir()
        }
    }
}
