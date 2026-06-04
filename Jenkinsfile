def branch = 'production'
def environmentName = branch == 'production' ? 'production' : 'staging'

pipeline {
    agent any
    environment {
        AWS_REGION  = 'us-east-1'
        REPO_URL    = 'git@github.com:daniunirdevops/todo-list-aws.git'
        BRANCH      = "${branch}"
        ENVIRONMENT = "${environmentName}"
        // to create a temporal bucket for deploy
        TEMP_BUCKET = "sam-temp-${env.BUILD_ID}-${ENVIRONMENT}"
    }
    stages {
        stage('Get Code') {
            steps {
                sh '''
                    # si esborro el workspace no caldria comprovar
                    git clone ${REPO_URL} .

                    # Ens assegurem que tenim develop actualitzat
                    git fetch origin
                    git checkout ${BRANCH}
                    git reset --hard origin/${BRANCH}
                '''                
                stash name: 'code', includes: '**'
            }
        }
    }
    // delete temporary bucket
    post {
        always {
            sh '''
                aws s3 rb s3://${TEMP_BUCKET} --force || true
            '''
            cleanWs() // clean all agents used
        }
    }    

}

