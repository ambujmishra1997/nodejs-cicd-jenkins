pipeline {

    agent any

    stages {

        stage('Checkout') {

            steps {

                checkout scm

                sh 'echo "Repository checked out successfully"'

                sh 'pwd'

                sh 'ls -la'
            }
        }
    }
}