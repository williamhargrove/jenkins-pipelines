node {
    timestamps() {

    // run tests in non-interactive mode
    //env.CI = true

        try {

        def remote = [:]
        remote.name = "stage-ui-1"
        remote.host = "stage-ui-1.REPLACE_ME.local"
        remote.allowAnyHosts = true


        slackSend color: 'good', message: "Build Started - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)", tokenCredentialId: 'Slack'

        stage('Preparation') {
            deleteDir()
        }

        stage('Checkout') {
            def scmVars = checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'amber_ui']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'Bitbucket', url: 'git@bitbucket.org:REPLACE_ME/amber_ui.git']]])
            env.COMMIT = scmVars.GIT_COMMIT.take(7)
        }

        stage('Dependancies') {
            nodejs('nodejs_8.12.0') {
                sh '''
                cd amber_ui
                npm install
                '''
            }

        }

        stage('Build') {
            nodejs('nodejs_8.12.0') {
                sh '''
                cd amber_ui
                npm run build
                '''
            }
        }

        /*
        stage('Test') {
            nodejs('nodejs_8.12.0') {
                sh '''
                cd amber_ui
                npm test
                '''
            }
        }
        */

        stage('Package') {
            sh '''
            cd amber_ui
            mv build build-${COMMIT}
            tar -czvf amber_ui-build-${COMMIT}.tgz build-${COMMIT}
            '''
        }

        stage('Archive') {
            archiveArtifacts artifacts: "amber_ui/amber_ui-build-${env.COMMIT}.tgz", onlyIfSuccessful: true, fingerprint: true
        }

        withCredentials([sshUserPrivateKey(credentialsId: 'Deploy', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) {
        remote.user = userName
        remote.identityFile = identity

        stage('Distribute: Staging') {
            // deploy user has sudo and write access to nginx docRoot
            //writeFile file: 'abc.sh', text: 'ls'
            //sshCommand remote: remote, command: 'ls -la /var/deploy'
            sshPut remote: remote, from: "amber_ui/amber_ui-build-${env.COMMIT}.tgz", into: '/var/www/ui'
            // run deploy script
            sshCommand remote: remote, command: "/var/www/ui/deploy.sh ${env.COMMIT}", sudo: true
            // restart nginx
            sshCommand remote: remote, command: "sudo systemctl restart nginx", sudo: true

            //sshGet remote: remote, from: 'abc.sh', into: 'bac.sh', override: true
            //sshScript remote: remote, script: 'abc.sh'
            //sshRemove remote: remote, path: 'abc.sh'
            }
        }


        } catch (err) {
            notifyFailed()
            throw err
        }
    }
}

def notifyFailed() {

    slackSend color: "danger", message: "Job: ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} failed", tokenCredentialId: 'Slack'
    emailext (
        attachLog: true,
        compressLog: false,
        from: 'build@REPLACE_ME.com',
        subject: "Failed: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
        to: "build@REPLACE_ME.com"
    )
}