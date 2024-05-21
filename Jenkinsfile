pipeline {
    agent any

    environment {
        BUCKET_NAME = 'aws-web-uno'
    }

    stages {
        stage('deploy to s3') {
            steps {
                echo 'Desplegando en S3 - INIT'
                withAWS(region: 'us-east-1', credentials: 'aws-credentials') {
                    sh 'aws s3 sync . s3://$BUCKET_NAME exclude "*.git*"'
                    sh 'aws s3 ls s3://$BUCKET_NAME'
                }
                echo 'Desplegando en S3 - END'
            }
        }
   }
}
