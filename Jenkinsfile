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
            }
        }

        stage('Static Test') {
            // test estático, bandit & flake8
            steps {
                echo "Run static tests..."

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
