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

            stage('Unit Test') {
                echo "Starting rabbitmq and postgres sidecar containers"
                docker.image('rabbitmq').withRun('-p 15672:15672/tcp -p 5671-5672:5671-5672/tcp') { rmq ->
                    /* Now start the postgres container*/
                    docker.image('database').withRun('-p 5432:5432/tcp -p 27017:27017/tcp') { pg ->
                    /* Wait until postgres is available */
                    sh 'while ! nc -z localhost 5432; do sleep 2; done'
                    sleep 5
                    sh '''
                        RBENV=~/rbenv
                        PATH=$PATH:${RBENV}/bin:${RBENV}/plugins/ruby-build/bin; export PATH
                        eval "$(rbenv init -)"
                        cd ${REPOSITORY}
                        bundle install
                        cp -p .env .env.test
                        # change the DB_NAME to docker version
                        sed -i 's#DB_NAME=rks_dev#DB_NAME=rks#' .env.test
                        sed -i 's#POSTGRES_USER=#POSTGRES_USER=rks#' .env.test
                        bundle exec rake db:test:prepare
                        chmod +x db/seed.rb
                        bundle exec db/seed.rb .env.test
                        RAILS_ENV=test bundle exec rspec --format RspecJunitFormatter --out rspec_results/results.xml
                      '''
                    }
                }
                echo "Archiving rspec results - Unit Test Step"
                junit '**/rspec_results/results.xml'
            }


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

