pipeline {
    agent any

    options {
        skipDefaultCheckout(true)   
    }

    stages {
        
        stage('Get Code') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/develop']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/daniunirdevops/todo-list-aws.git'
                    ]]
                ])
                stash name: 'code', includes: '*/**'
            }
        }


        stage('Static Test') {
            steps {
                echo "Run static tests..."
                // obtain code from previous stage
                unstash 'code'     
                sh '''
                    export PYTHONPATH=$WORKSPACE
                    bandit \
                            -r src/*.py \
                            -f custom \
                            -o bandit.out \
                            --msg-template "{abspath}:{line}: {severity}: {msg}"
                '''                       
                sh '''
                    export PYTHONPATH=$WORKSPACE
                    flake8 --format=pylint --exit-zero --output-file=flake8.out src

                '''

                // no quality gates, just report the issues
                recordIssues(
                    tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], 
                )                
                recordIssues (
                    tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], 
                    )

                }
        }

        stage('Deploy') {
            steps {
                echo "Desplegando desde develop..."
                // sh './scripts/deploy.sh'
            }
        }

        stage('Rest Test') {

            steps {
                echo "Run REST tests..."

            }
        }
        stage('Promote to Prod') {
            when {
                branch 'develop'
            }

            steps {
                echo "Merge branch develop with master..."

            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
        }
    }
}
