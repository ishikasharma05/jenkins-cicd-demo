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

        stage('Install Dependencies') {
            steps {
                echo 'Installing requirements...'
                sh '''
                    python3 --version
                    python3 -m pip install --upgrade pip --break-system-packages
                    python3 -m pip install -r requirements.txt --break-system-packages
                '''
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