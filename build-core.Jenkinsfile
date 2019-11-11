node {
    timestamps(){

        try{

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
            def scmVars = checkout([$class: 'GitSCM', branches: [[name: '*/example']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'SubmoduleOption', disableSubmodules: false, parentCredentials: true, recursiveSubmodules: false, reference: '', trackingSubmodules: true]], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'Bitbucket', url: 'git@bitbucket.org:example/core.git']]])
            env.COMMIT = scmVars.GIT_COMMIT.take(7)
        }

        stage('Build') {
            sh 'PIE=1 make -j2 all'
            sh 'make -j2 tests'
        }
/*
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
*/
        } catch (err) {
            notifyFailed()
            throw err
        }
    }
}

def notifyFailed() {
    emailext (
        attachLog: true,
        compressLog: true,
        from: 'build@example.com',
        subject: "Failed: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
        to: "build@example.com"
    )
}
