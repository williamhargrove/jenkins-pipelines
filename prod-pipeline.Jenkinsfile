node ('build-ruby') {

    timestamps() {
        try {

            env.REPOSITORY = 'bdprocessor'
            def COMMIT_HASH = null
            def CONFIRM_COMMIT_HASH = null
            echo "${env.REPOSITORY}"
            echo "${env.REPOSITORY_NAME}"

            stage('Preparation') {
                deleteDir()
            }

            stage ('Enter commit to deploy to production') {
                COMMIT_HASH = input message: 'Enter commit to deploy:', parameters: [string(defaultValue: 'HEAD', description: 'Commit to deploy to production', name: 'COMMIT_HASH', trim: true)]

                println(COMMIT_HASH);
            }

            stage ('Confirm commit to deploy to production') {
                CONFIRM_COMMIT_HASH = input message: 'Confirm commit to deploy:', parameters: [string(defaultValue: 'HEAD', description: 'Confirm commit to deploy to production', name: 'CONFIRM_COMMIT_HASH', trim: true)]

                println(CONFIRM_COMMIT_HASH)

            }

            if (COMMIT_HASH == CONFIRM_COMMIT_HASH) {

            stage('Checkout') {
                def scmVars = checkout([$class: 'GitSCM', branches: [[name: COMMIT_HASH.take(7)]], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: "${env.REPOSITORY}"]], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'Bitbucket', url: "git@bitbucket.org:REPLACE_ME/${env.REPOSITORY}.git"]]])
                env.COMMIT = COMMIT_HASH.take(7)
            }

//            prod('Unit Test - Dependancies') {
//                echo "Starting mongodb sidecar containers"
//                docker.image('database').withRun('-p 5432:5432/tcp -p 27017:27017/tcp') { c ->
//                   /* Wait until mongo is available */
//                   sh 'while ! nc -z localhost 27017; do sleep 1; done'
//                   sh '''
//                      RBENV=~/rbenv
//                      PATH=$PATH:${RBENV}/bin:${RBENV}/plugins/ruby-build/bin; export PATH
//                      eval "$(rbenv init -)"
//                      cd ${REPOSITORY}
//                      bundle install
//                      rspec
//                      //RAILS_ENV=test bundle exec rspec spec --format html --out rspec_results/results.html --format RspecJunitFormatter --out rspec_results/results.xml
//                   '''
//                }
//            }


            stage('Build Production') {
                env.ENVIRONMENT='cflex_prod'
                echo "Initialise RBENV"
                // git tag "${env.ENVIRONMENT}"-"${env.REPOSITORY}"-$(date +"%Y%m%d_%H%M%S") "${env.COMMIT}"
                // git push --tags
                sh '''
                    RBENV=~/rbenv
                    PATH=$PATH:${RBENV}/bin:${RBENV}/plugins/ruby-build/bin; export PATH
                    eval "$(rbenv init -)"

                    cd ${REPOSITORY}
                    bundle install --deployment --without development test
                    # copy config files
                    cp -rf ~/configuration.${ENVIRONMENT}/${REPOSITORY}/ ../

                    # Output the tags into a file in the repo directory.
                    echo "Repo:" ${REPOSITORY}, "Target:" ${ENVIRONMENT} > version
                    #echo "Tags:" $(git describe --tags) >> version
                    echo "Commit:" ${COMMIT} >> version
                    echo "Repo clone date:" $timestamp >> version

                    # Remove the .git directory
                    rm -rf .git
                '''
            }

            stage('Package Production') {
                echo "Package artifact into a deploy"
                sh '''
                    tar -czf ${REPOSITORY}.${ENVIRONMENT}.tgz ${REPOSITORY}
                    cp -f ${REPOSITORY}.${ENVIRONMENT}.tgz ~/downloads
                '''
            }

             stage('Distribute Production') {
                echo "Copy artifact to destination servers"
                sh '''
                    for host in prod-bdp; do
                    scp ~/downloads/${REPOSITORY}.${ENVIRONMENT}.tgz ruby@${host}:${REPOSITORY}.tgz
                    #scp ~/downloads/${REPOSITORY}.${ENVIRONMENT}.tgz.md5sum ruby@${host}:${REPOSITORY}.tgz.md5sum
                    done
                '''
            }

            stage('Deploy Production') {
                echo "Deploy artifact"
                sh '''
                ssh ruby@prod-bdp ". .bash_profile; ~/${REPOSITORY}.sh"
                '''
                def USER = wrap([$class: 'BuildUser']) {
                    return env.BUILD_USER
                }
                slackSend color: 'good', message: "Deployed ${env.REPOSITORY} ${env.COMMIT} by ${USER} to Production.", tokenCredentialId: 'Slack', channel: '#REPLACE_ME_live'
            }
        } else {
            println ('Commit hashes do not match')

        }
        } catch (err) {

        throw err

        }

    }

}
