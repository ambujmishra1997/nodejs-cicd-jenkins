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


        stage('Integration Test') {

            steps {

                dir('src') {

                    sh '''
                        echo "Starting application..."

                        npm start > app.log 2>&1 &
                        APP_PID=$!

                        echo "Waiting for application..."

                        for i in 1 2 3 4 5 6 7 8 9 10
                        do
                            if curl -f http://localhost:3000
                            then
                                echo "Application is ready"
                                break
                            fi

                            if [ "$i" = "10" ]
                            then
                                echo "Application failed to start"
                                cat app.log
                                kill $APP_PID || true
                                exit 1
                            fi

                            sleep 2
                        done

                        echo "Running integration tests..."

                        npm test
                        TEST_STATUS=$?

                        echo "Stopping application..."
                        kill $APP_PID || true

                        exit $TEST_STATUS
                    '''
                }
            }
        }

    }
}