pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Pulling source code from GitHub...'
                checkout scm
            }
        }



        stage('Run Tests') {
            steps {
                echo 'Running unit tests...'
                sh '''
                    python3 -m pytest -v
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deployment stage reached.'
            }
        }
    }

    post {
        always {
            echo 'Pipeline run finished.'
        }

        success {
            echo 'All tests passed successfully!'
        }

        failure {
            echo 'Build or tests failed.'
        }
    }
}