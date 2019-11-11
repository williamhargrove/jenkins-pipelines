node {
    timestamps(){

        try{

        stage('Preparation') {
            deleteDir()
        }

        stage('Checkout') {
            def scmVars = checkout([$class: 'GitSCM', branches: [[name: '*/example']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'example_api']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'Bitbucket', url: 'git@bitbucket.org:example/example_api.git']]])
            env.COMMIT = scmVars.GIT_COMMIT.take(7)
        }

        stage('Build') {
            sh 'cd example_api; mvn package -DskipTests'
        }

        stage('Archive') {
            archiveArtifacts artifacts: "example_api/target/api.jar", onlyIfSuccessful: true, fingerprint: true
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
