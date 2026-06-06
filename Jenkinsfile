pipeline {
    agent any
    environment {
        REPO_URL    = 'https://github.com/daniunirdevops/todo-list-aws.git'
        AWS_REGION  = "us-east-1"
        BRANCH      = "master"
        ENVIRONMENT = "production"
        // to create a temporal bucket for deploy
        TEMP_BUCKET = "sam-temp-${env.BUILD_ID}-${ENVIRONMENT}"
    }
    stages {
        stage('Get Code') {
            steps {
                sh 'whoami ; hostname'
                git branch: env.BRANCH,
                    url: env.REPO_URL                
                stash name: 'code', includes: '**'
            }
        }
        stage('Deploy') {
            steps {
                sh 'whoami ; hostname'
                unstash 'code'     
                sh '''
                    aws s3 mb s3://${TEMP_BUCKET} --region ${AWS_REGION}
                    sam build
                    sam validate --region ${AWS_REGION}

                    sam deploy template.yaml \
                        --config-env ${ENVIRONMENT} \
                        --region ${AWS_REGION} \
                        --s3-bucket ${TEMP_BUCKET} \
                        --no-confirm-changeset \
                        --force-upload \
                        --no-fail-on-empty-changeset \
                        --on-failure DELETE \
                        --no-progressbar
                '''
                // obtain the URL
                script {
                    env.BASE_URL = sh(
                        script: """
                            aws cloudformation describe-stacks \
                                --stack-name todo-list-aws-${ENVIRONMENT} \
                                --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" \
                                --region ${AWS_REGION} \
                                --output text
                        """,
                        returnStdout: true
                    ).trim()   
                    echo "BASE_URL = ${env.BASE_URL}"
                }
            }
            post {
                always {
                    sh '''
                        aws s3 rb s3://${TEMP_BUCKET} --force || true
                    '''
                }
            }    
        }
        stage('Rest Test') {
            steps {
                sh 'whoami ; hostname'
                // obtain environment variable and assign to shell variable 
                // we run test readonly method
                sh """
                    export BASE_URL="${env.BASE_URL}"
                    pytest -s \
                        test/integration/todoApiTest.py::TestApi::test_api_listtodos \
                        test/integration/todoApiTest.py::TestApi::test_api_gettodo \
                        --junitxml=result-rest.xml
                """
                // show result
                junit 'result-rest.xml'            
            }
        }
     }
    
    post {
        always {
            cleanWs() // clean all agents used
        }
    }  
    
}
