node {
    timestamps() {

        try {

        // run tests in non-interactive mode
        env.CI = true

        slackSend color: 'good', message: "Build Started - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)", tokenCredentialId: 'Slack'

        stage('Preparation') {
            deleteDir()
        }

        stage('Checkout') {
            def scmVars = checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'slp-express']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'Bitbucket', url: 'git@bitbucket.org:example/slp-express.git']]])
            env.COMMIT = scmVars.GIT_COMMIT.take(7)
        }

        stage('Dependancies') {
            nodejs('nodejs_10.16.1') {
                sh '''
                cd slp-express
                npm install
                '''
            }

        }

        /*
        stage('Build') {
            nodejs('nodejs_10.16.1') {
                sh '''
                cd slp-express
                npm run build
                '''
            }
        }
        */

        stage('Test') {
            nodejs('nodejs_10.16.1') {
                timeout(activity: true, time: 2, unit: 'MINUTES') {
                    sh '''
                    cd slp-express
                    npm test
                    '''
                }
            }
        }

        /*
        stage('Package') {
            sh 'cd amber_ui; tar -czvf amber_ui-build-${COMMIT}.tgz build'
        }
        */

        /*
        stage('Archive') {
            archiveArtifacts artifacts: "amber_ui/amber_ui-build-${env.COMMIT}.tgz", onlyIfSuccessful: true, fingerprint: true
        }
        */

        slackNotifier(currentBuild.currentResult)

        } catch (err) {
            notifyFailed()
            throw err
        }
    }
}

def notifyFailed() {
    emailext (
        attachLog: true,
        compressLog: false,
        from: 'build@example.com',
        subject: "Failed: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
        to: "build@example.com"
    )
}

// move the function below to a GIT hosted shared library
def slackNotifier (String buildResult) {
  if ( buildResult == "SUCCESS" ) {
    slackSend color: "good", message: "Job: ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} was successful"
  }
  else if( buildResult == "FAILURE" ) {
    slackSend color: "danger", message: "Job: ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} failed"
  }
  else if( buildResult == "UNSTABLE" ) {
    slackSend color: "warning", message: "Job: ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} is unstable"
  }
  else {
    slackSend color: "danger", message: "Job: ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} result was unclear"
  }
}