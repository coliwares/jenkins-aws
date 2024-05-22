/* groovylint-disable CompileStatic, NoDef, SpaceAfterClosingBrace, UnnecessaryGetter, VariableTypeRequired */
def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger'
]

def getBuildUser() {
    def buildUser = currentBuild.rawBuild.getCause(Cause.UserIdCause).getUserId()
    if (buildUser == null) {
        result = 'anonymous'
    } else {
        result = buildUser
    }
    return result
}

pipeline {
    agent any
    environment {
        BUCKET = 'aws-web-angular'
        BUILD_USER = ''
    }
    stages {
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
    }
    post {
        always {
            echo 'This will always run'
            script {
                BUILD_USER = getBuildUser()
            }
            slackSend ( color: COLOR_MAP[currentBuild.currentResult], 
                        message: "Build *${currentBuild.currentResult}* Job ${env.JOB_NAME} build #${env.BUILD_NUMBER} by ${BUILD_USER} \n in stage ${env.STAGE_NAME} \n more info at ${env.BUILD_URL}")
        }
    }
}
