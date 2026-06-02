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
                stash name: 'code', includes: 'app/**, test/**, pytest.ini'
            }
        }

        stage('Show Branch') {
            steps {
                echo "BRANCH_NAME=${env.BRANCH_NAME}"
                echo "GIT_BRANCH=${env.GIT_BRANCH}"
                sh 'git branch --show-current'
            }
        }

        stage('Static Test') {
            // test estático, bandit & flake8
            when {
                branch 'develop'
            }            
            steps {
                echo "Run static tests..."
                // obtain code from previous stage
                unstash 'code'     
                sh '''
                    export PYTHONPATH=$WORKSPACE
                    python3 -m bandit \
                            --exit-zero \
                            -r src/*.py \
                            -f custom \
                            -o bandit.out \
                            --msg-template "{abspath}:{line}: {severity}: {msg}"
                '''                       
                sh 'cat bandit.out'
                sh '''
                    export PYTHONPATH=$WORKSPACE
                    python3 -m flake8 --format=pylint --exit-zero --output-file=flake8.out src

                '''
                sh 'cat flake8.out'

                // no quality gates, just report the issues
                recordIssues(
                    tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], 
                    // qualityGates: [
                    //     [threshold: 2, type: 'TOTAL', unstable: true],
                    //     [threshold: 4, type: 'TOTAL', unstable: false]
                    // ]
                )                
                recordIssues (
                    tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], 
                    // qualityGates : [
                    //     [threshold:8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unstable: false]
                    //     ]
                    )

                }
        }

        stage('Deploy') {
            when {
                branch 'develop'
            }
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
