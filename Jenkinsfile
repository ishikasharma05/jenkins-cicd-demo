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

        stage('Setup Python') {
            steps {
                echo 'Creating Python virtual environment...'
                sh '''
                    python3 -m venv .venv
                    . .venv/bin/activate
                    python -m pip install --upgrade pip
                    python -m pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running unit tests...'
                sh '''
                    . .venv/bin/activate
                    python -m pytest -v
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                echo 'Deployment steps will be added later.'
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
