pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID     = 'AKIARBSZNU3DYFI423EZ'
        AWS_SECRET_ACCESS_KEY = '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'
            AWS_DEFAULT_REGION    = 'us-east-1'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }

        stage('Terraform Validate') {
            steps {
                sh 'terraform validate'
            }
        }

        stage('Terraform Plan') {
            steps {
                sh '''
                    terraform plan -out=tfplan
                    terraform show -no-color tfplan > terraform-plan.txt
                '''
            }
        }

        stage('Archive Plan') {
            steps {
                archiveArtifacts artifacts: 'terraform-plan.txt', fingerprint: true
                stash includes: 'tfplan', name: 'tfplan'
            }
        }

        stage('Manual Approval') {
            steps {
                timeout(time: 30, unit: 'MINUTES') {
                    input(
                        message: 'Review terraform plan artifact and approve deployment',
                        ok: 'Deploy'
                    )
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                unstash 'tfplan'
                sh 'terraform apply -auto-approve tfplan'
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully'
        }

        failure {
            echo 'Deployment failed'
        }
    }
}
