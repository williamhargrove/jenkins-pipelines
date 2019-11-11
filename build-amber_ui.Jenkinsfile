node {
    timestamps() {

    // run tests in non-interactive mode
    // env.CI = true

        try{

        stage('Preparation') {
            deleteDir()
        }

        stage('Checkout') {
            def scmVars = checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'amber_ui']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'Bitbucket', url: 'git@bitbucket.org:example/amber_ui.git']]])
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
            sh 'cd amber_ui; tar -czvf amber_ui-build-${COMMIT}.tgz build'
        }

        stage('Archive') {
            archiveArtifacts artifacts: "amber_ui/amber_ui-build-${env.COMMIT}.tgz", onlyIfSuccessful: true, fingerprint: true
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
        from: 'build@example.com',
        subject: "Failed: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
        to: "build@example.com"
    )
}
