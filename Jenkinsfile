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


        stage('Install Dependencies') {
            steps {
                dir('src') {
                    sh 'node --version'
                    sh 'npm --version'
                    sh 'npm ci'
                }
            }
        }


        stage('Lint') {
            steps {
                dir('src') {
                    sh 'npm run lint'
                }
            }
        }

    }
}