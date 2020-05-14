node {
    timestamps(){

        try{

        def stage_1 = [:]
        stage_1.name = "stage-apiui-1"
        stage_1.host = "stage-apiui-1.REPLACE_ME.local"
        stage_1.allowAnyHosts = true

        def prod_1 = [:]
        prod_1.name = "prod-apiui-1"
        prod_1.host = "prod-apiui-1.REPLACE_ME.local"
        prod_1.allowAnyHosts = true

        def prod_2 = [:]
        prod_2.name = "prod-apiui-2"
        prod_2.host = "prod-apiui-2.REPLACE_ME.local"
        prod_2.allowAnyHosts = true

        slackSend color: 'good', message: "Build Started - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)", tokenCredentialId: 'Slack'

        stage('Preparation') {
            deleteDir()
        }

        slackSend color: 'good', message: "Checking out SCM code for ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)", tokenCredentialId: 'Slack'
        stage('Checkout') {
            def scmVars = checkout([$class: 'GitSCM', branches: [[name: '*/REPLACE_ME']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'REPLACE_ME_api']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'Bitbucket', url: 'git@bitbucket.org:REPLACE_ME/REPLACE_ME_api.git']]])
            env.COMMIT = scmVars.GIT_COMMIT.take(7)
        }

        slackSend color: 'good', message: "Started Maven build for ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)", tokenCredentialId: 'Slack'
        stage('Build') {
            sh 'cd REPLACE_ME_api; mvn package -DskipTests'
        }

        slackSend color: 'good', message: "Archiving build artifact for ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)", tokenCredentialId: 'Slack'
        stage('Archive') {
            archiveArtifacts artifacts: "REPLACE_ME_api/target/api.jar", onlyIfSuccessful: true, fingerprint: true
        }

        if (currentBuild.currentResult == 'SUCCESS') {
            slackSend color: 'good', message: "Build result success for ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)", tokenCredentialId: 'Slack'
            stage('Deploy: Staging') {

            slackSend color: 'good', message: "Deploy to Staging for ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)", tokenCredentialId: 'Slack'
            withCredentials([sshUserPrivateKey(credentialsId: 'Deploy', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) {
                stage_1.user = 'java'
                stage_1.identityFile = identity

                sshCommand remote: stage_1, command: "mkdir /home/java/apiui-${COMMIT}"
                sshPut remote: stage_1, from: "REPLACE_ME_api/target/api.jar", into: "/home/java/apiui-${COMMIT}/api.jar"

                // run deploy script
                sshCommand remote: stage_1, command: "/home/java/deploy.sh ${env.COMMIT}"
                }
            }

            stage('Deploy: Production') {
            slackSend color: 'good', message: "Deploy to Production for ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)", tokenCredentialId: 'Slack'
            withCredentials([sshUserPrivateKey(credentialsId: 'Deploy', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) {
                prod_1.user = 'java'
                prod_1.identityFile = identity
                prod_2.user = 'java'
                prod_2.identityFile = identity

                sshCommand remote: prod_1, command: "mkdir /home/java/apiui-${COMMIT}"
                sshCommand remote: prod_2, command: "mkdir /home/java/apiui-${COMMIT}"
                sshPut remote: prod_1, from: "REPLACE_ME_api/target/api.jar", into: "/home/java/apiui-${COMMIT}/api.jar"
                sshPut remote: prod_2, from: "REPLACE_ME_api/target/api.jar", into: "/home/java/apiui-${COMMIT}/api.jar"

                // run deploy script
                sshCommand remote: prod_1, command: "/home/java/deploy.sh ${env.COMMIT}"
                sleep 5
                sshCommand remote: prod_2, command: "/home/java/deploy.sh ${env.COMMIT}"
                }
            }

        } else {
        slackSend color: 'warning', message: "Build unsuccessful ${currentBuild.currentResult} for ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)", tokenCredentialId: 'Slack'
                }

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
        from: 'build@REPLACE_ME.com',
        subject: "Failed: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
        to: "build@REPLACE_ME.com"
    )
}

