node {
    timestamps(){

        try {

        slackSend "Build Started - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"

        stage('Preparation') {
            deleteDir()
        }

        /*
        stage('Dependancies') {
            dir("/var/lib/jenkins/dependancies/jar") {
            fileOperations([fileCopyOperation(excludes: '', flattenFiles: true, includes: '*.jar', targetLocation: "${WORKSPACE}")])
            }
        }
        */

        stage('Checkout') {
            def scmVars = checkout([$class: 'GitSCM', branches: [[name: '*/example']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'rks']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'Bitbucket', url: 'git@bitbucket.org:example/rks.git']]])
            env.COMMIT = scmVars.GIT_COMMIT.take(7)
            env.STARTTIMEINMILLIS = currentBuild.startTimeInMillis
            sh '''
            git tag ci_build-rks-${STARTTIMEINMILLIS} ${COMMIT}
            git push --tags

            '''
        }

        stage('Build') {
            sh 'make -C mattwhitlock-common && ln -fn mattwhitlock-common/target/*.jar ./'
            sh 'make -C mattwhitlock-ognl && ln -fn mattwhitlock-ognl/target/*.jar ./'
            sh 'make -C java-library && ln -fn java-library/target/*.jar ./'
            sh 'make -C java-common && ln -fn java-common/target/*.jar ./'
            sh 'make -C apiproxy && ln -fn apiproxy/target/*.jar ./'
        }

        stage('Package') {
            sh 'mkdir -p apiproxy-${COMMIT}/log'
            sh 'cp -p *.jar apiproxy-${COMMIT}'
            sh 'echo "Commit:" ${COMMIT} > apiproxy-${COMMIT}/version'
            sh 'echo "Repo clone date:" ${BUILD_TAG} >> apiproxy-${COMMIT}/version'
            sh 'tar -cvf apiproxy-${COMMIT}.tgz apiproxy-${COMMIT}'
        }

        stage('Archive') {
            archiveArtifacts artifacts: "apiproxy-${env.COMMIT}.tgz", onlyIfSuccessful: true, fingerprint: true
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
