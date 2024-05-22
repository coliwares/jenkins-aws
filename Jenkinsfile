/* groovylint-disable CompileStatic, NoDef, SpaceAfterClosingBrace, UnnecessaryGetter, VariableTypeRequired */
def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger'
]

def getBuildUser() {
    //obtener usuario ejecutor de job de jenkins
    def buildUser = ''
    def buildCauses = currentBuild.rawBuild.getCauses()
    for (cause in buildCauses) {
        if (cause.getShortDescription().contains('Started by user')) {
            buildUser = cause.getShortDescription().split(' ')[3]
        }
    }
    return buildUser
}

//obtener el stage que falló
def getFailedStage() {
    def failedStages = currentBuild.rawBuild.getAction(hudson.model.CauseAction).getCauses().get(0).getShortDescription().split(' ')[0]
    def log = ''
    def causes = currentBuild.rawBuild.getAction(hudson.model.CauseAction).getCauses()
    for (cause in causes) {
        log += cause.toString() + '\n'
    }
    //imprime el stage que falló
    echo "Failed stage: ${log}"

    for (stage in failedStages) {
        failedStage = stage
    }
    return failedStage
}



pipeline {
    agent any
    environment {
        BUCKET = 'aws-web-angular'
        BUILD_USER = ''
        STAGE_FAILED = ''
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
                STAGE_FAILED = getFailedStage()
            }
            slackSend   color: COLOR_MAP[currentBuild.currentResult], 
                        message: "Build *${currentBuild.currentResult}* Job ${env.JOB_NAME} build #${env.BUILD_NUMBER} by ${BUILD_USER} \n in stage ${STAGE_FAILED} \n more info at ${env.BUILD_URL}"
        }
    }
}
