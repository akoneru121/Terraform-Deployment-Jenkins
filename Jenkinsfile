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

        // Only AWS credentials needed now
        AWS_CREDENTIALS = credentials('aws-credentials')
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

        stage('Terraform Init (Local State)') {
            steps {
                dir('test-3-terraform') {
                    sh 'terraform init'
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
