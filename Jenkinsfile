
pipeline {
    agent any

    stages {

        stage('Initialisation') {
            steps {
                echo 'Starting DevSecOps Capstone Pipeline'
                echo 'Preparing Jenkins environment'
            }
        }

        stage('Environment Check') {
            steps {
                sh '''
                    echo "Checking Jenkins environment..."
                    java -version
                    git --version
                    docker --version
                '''
            }
        }

        stage('Repository Verification') {
            steps {
                sh '''
                    echo "Checking project repository..."
                    pwd
                    ls -la
                '''
            }
        }

        stage('Pipeline Completed') {
            steps {
                echo 'Initial DevSecOps pipeline completed successfully!'
            }
        }
    }

    post {
        success {
            echo 'Pipeline execution: SUCCESS'
        }

        failure {
            echo 'Pipeline execution: FAILURE'
        }
    }
}
