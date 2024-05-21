pipeline {
    agent any
    environment {
        BUCKET = 'aws-web-angular'
    }

    stages {
        try {
            stage('init') {
                agent {
                    docker {
                        image 'node:erbium-alpine'
                        args '-u root:root'
                    }
                }
                steps {
                    sh 'npm install'
                }
            }
            stage('test') {
                agent {
                    docker {
                        image 'roxsross12/node-chrome'
                        args '-u root:root'
                    }
                }
                steps {
                    sh 'npm run test'
                }
            }
            stage('build') {
                agent {
                    docker {
                        image 'node:erbium-alpine'
                        args '-u root:root'
                    }
                }
                steps {
                    sh 'npm run build'
                    stash includes: 'dist/**/**', name: 'dist'
                }
            }
            stage('deploy') {
                steps {
                    withAWS(credentials: 'aws-credentials', region: 'us-east-1') {
                        unstash 'dist'
                        sh 'aws s3 sync dist/. s3://$BUCKET --exclude ".git/*"'
                        sh 'aws s3 ls s3://$BUCKET '
                    }
                }
            }
        } catch (e) {
            currentBuild.result = 'FAILURE'
            slackSend(
                color: '#FF0000',
                message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
            )

            throw e
        }
    }
}
