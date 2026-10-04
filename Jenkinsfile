pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }


        stage('Install Dependencies') {
            steps {
                dir('src') {
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
                        npm start > app.log 2>&1 &
                        APP_PID=$!

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

                        npm test
                        TEST_STATUS=$?

                        kill $APP_PID || true

                        exit $TEST_STATUS
                    '''
                }
            }
        }


        stage('Security Scan') {

            steps {

                sh '''
                    echo "Running Trivy security scan..."

                    trivy --config "" fs . \
                    --severity HIGH,CRITICAL \
                    --ignore-unfixed \
                    --format json \
                    --output trivy.result.json \
                    --exit-code 1
                '''
            }

            post {

                always {

                    archiveArtifacts artifacts: 'trivy.result.json',
                                     allowEmptyArchive: true
                }
            }
        }

    }
}