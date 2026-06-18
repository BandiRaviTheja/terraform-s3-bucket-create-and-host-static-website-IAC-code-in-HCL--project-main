pipeline {
agent any

environment {
    AWS_REGION = 'us-east-1'
}

stages {

    stage('Verify Tools') {
        steps {
            sh 'terraform --version'
            sh 'aws --version'
        }
    }

    stage('Terraform Init') {
        steps {
            withAWS(credentials: 'aws-cred', region: "${AWS_REGION}") {
                sh 'terraform init'
            }
        }
    }

    stage('Terraform Validate') {
        steps {
            withAWS(credentials: 'aws-cred', region: "${AWS_REGION}") {
                sh 'terraform validate'
            }
        }
    }

    stage('Terraform Apply') {
        steps {
            withAWS(credentials: 'aws-cred', region: "${AWS_REGION}") {
                sh 'terraform apply -auto-approve'
            }
        }
    }

    stage('Upload Website Files to S3') {
        steps {
            withAWS(credentials: 'aws-cred', region: "${AWS_REGION}") {
                sh '''
                    BUCKET_NAME=$(terraform output -raw name | cut -d'.' -f1)

                    echo "Uploading website files to bucket: $BUCKET_NAME"

                    aws s3 sync . s3://$BUCKET_NAME \
                      --exclude ".git/*" \
                      --exclude ".terraform/*" \
                      --exclude ".terraform.lock.hcl" \
                      --exclude "*.tf" \
                      --exclude "*.hcl" \
                      --exclude "Jenkinsfile" \
                      --exclude "*.md"
                '''
            }
        }
    }
}

post {
    success {
        echo 'Static website deployment successful'
        sh 'terraform output'
    }

    failure {
        echo 'Static website deployment failed'
    }

    always {
        cleanWs()
    }
}

}
