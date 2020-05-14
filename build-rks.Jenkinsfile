node ('build-ruby') {

    timestamps() {
        try {

            env.REPOSITORY = 'rks'
            echo "{env.projectName}"

            stage('Preparation') {
                deleteDir()
            }

            stage('Checkout') {
                def scmVars = checkout([$class: 'GitSCM', branches: [[name: '*/REPLACE_ME']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: "${env.REPOSITORY}"]], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'Bitbucket', url: "git@bitbucket.org:REPLACE_ME/${env.REPOSITORY}.git"]]])
                env.COMMIT = scmVars.GIT_COMMIT.take(7)
            }

//            stage('Unit Test - Dependancies') {
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


            stage('Build Staging') {
                env.ENVIRONMENT='cflex_stage'
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


        } catch (err) {

        throw err

        }

    }

}

//timeout(time: 3, unit: 'HOURS') {
//    input message: 'Do you want to approve the deploy into production?', parameters: [booleanParam(defaultValue: false, description: '', name: 'PROCEED')]
//}

