pipeline {
    agent {
        docker {
            image 'python:3.12-slim'
        }
    }

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

        stage('Install Dependencies') {
            steps {
                echo 'Installing requirements...'
                sh '''
                    python --version
                    python -m pip install --upgrade pip
                    python -m pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running unit tests...'
                sh '''
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