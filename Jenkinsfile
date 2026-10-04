pipeline {

    agent any

    stages {

        // =====================================================
        // STAGE 1: CHECKOUT SOURCE CODE
        // =====================================================
        // Pulls the latest source code from the configured
        // Git repository into the Jenkins workspace.
        // =====================================================

        stage('Checkout') {

            steps {

                checkout scm

                sh '''
                    echo "======================================"
                    echo " Repository checked out successfully"
                    echo "======================================"

                    pwd
                    ls -la
                '''
            }
        }


        // =====================================================
        // STAGE 2: INSTALL DEPENDENCIES
        // =====================================================
        // Installs Node.js dependencies using package-lock.json.
        // npm ci is preferred in CI/CD because it provides
        // clean and reproducible dependency installation.
        // =====================================================

        stage('Install Dependencies') {

            steps {

                dir('src') {

                    sh '''
                        echo "======================================"
                        echo " Installing Node.js dependencies"
                        echo "======================================"

                        node --version
                        npm --version

                        npm ci
                    '''
                }
            }
        }


        // =====================================================
        // STAGE 3: CODE QUALITY / LINT
        // =====================================================
        // Runs the linting rules configured in the Node.js
        // application.
        //
        // If linting fails, Jenkins stops the pipeline.
        // =====================================================

        stage('Lint') {

            steps {

                dir('src') {

                    sh '''
                        echo "======================================"
                        echo " Running code linting"
                        echo "======================================"

                        npm run lint
                    '''
                }
            }
        }


        // =====================================================
        // STAGE 4: INTEGRATION TEST
        // =====================================================
        // Starts the Node.js application on port 3000.
        // Jenkins waits until the application responds,
        // then executes the integration tests.
        //
        // The application is stopped after testing.
        // =====================================================

        stage('Integration Test') {

            steps {

                dir('src') {

                    sh '''
                        echo "======================================"
                        echo " Starting application"
                        echo "======================================"

                        npm start > app.log 2>&1 &

                        APP_PID=$!

                        echo "Application PID: $APP_PID"
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

                            echo "Waiting..."
                            sleep 2
                        done


                        echo "======================================"
                        echo " Running integration tests"
                        echo "======================================"

                        set +e

                        npm test

                        TEST_STATUS=$?

                        set -e


                        echo "======================================"
                        echo " Stopping application"
                        echo "======================================"

                        kill $APP_PID || true


                        exit $TEST_STATUS
                    '''
                }
            }
        }


        // =====================================================
        // STAGE 5: TRIVY SECURITY SCAN
        // =====================================================
        // Scans the repository and dependency metadata for
        // HIGH and CRITICAL vulnerabilities.
        //
        // --exit-code 1 acts as a security gate.
        // If unacceptable vulnerabilities are found,
        // the pipeline stops before Docker image creation.
        //
        // The JSON report is archived in Jenkins.
        // =====================================================

        stage('Security Scan') {

            steps {

                sh '''
                    echo "======================================"
                    echo " Running Trivy security scan"
                    echo "======================================"

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

                    archiveArtifacts(
                        artifacts: 'trivy.result.json',
                        allowEmptyArchive: true
                    )
                }
            }
        }


        // =====================================================
        // STAGE 6: BUILD DOCKER IMAGE
        // =====================================================
        // Builds the production Docker image using the
        // Dockerfile stored in the repository root.
        //
        // The Git commit ID is used as the Docker image tag
        // to provide traceability.
        // =====================================================

        stage('Build Docker Image') {

            steps {

                sh '''
                    echo "======================================"
                    echo " Building Docker image"
                    echo "======================================"

                    docker build \
                    -t nodejs-demoapp:${GIT_COMMIT} \
                    .


                    echo "======================================"
                    echo " Docker image created successfully"
                    echo "======================================"

                    docker images nodejs-demoapp
                '''
            }
        }


        // =====================================================
        // STAGE 7: PUBLISH IMAGE TO DOCKER HUB
        // =====================================================
        // Retrieves Docker Hub credentials securely from
        // Jenkins Credentials.
        //
        // The same image receives:
        // 1. Git commit SHA tag
        // 2. latest tag
        //
        // Both tags are pushed to Docker Hub.
        // =====================================================

        stage('Publish to Docker Hub') {

            steps {

                withCredentials([

                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKERHUB_USERNAME',
                        passwordVariable: 'DOCKERHUB_TOKEN'
                    )

                ]) {

                    sh '''
                        echo "======================================"
                        echo " Logging in to Docker Hub"
                        echo "======================================"

                        echo "$DOCKERHUB_TOKEN" | \
                        docker login \
                        -u "$DOCKERHUB_USERNAME" \
                        --password-stdin


                        echo "======================================"
                        echo " Tagging Docker image"
                        echo "======================================"

                        docker tag \
                        nodejs-demoapp:${GIT_COMMIT} \
                        $DOCKERHUB_USERNAME/nodejs-demoapp:${GIT_COMMIT}


                        docker tag \
                        nodejs-demoapp:${GIT_COMMIT} \
                        $DOCKERHUB_USERNAME/nodejs-demoapp:latest


                        echo "======================================"
                        echo " Pushing Docker image"
                        echo "======================================"

                        docker push \
                        $DOCKERHUB_USERNAME/nodejs-demoapp:${GIT_COMMIT}


                        docker push \
                        $DOCKERHUB_USERNAME/nodejs-demoapp:latest


                        echo "======================================"
                        echo " Docker image published successfully"
                        echo "======================================"

                        docker logout
                    '''
                }
            }
        }

    }


    // =========================================================
    // PIPELINE RESULT
    // =========================================================

    post {

        success {

            echo '======================================'
            echo ' CI/CD PIPELINE COMPLETED SUCCESSFULLY'
            echo '======================================'
        }


        failure {

            echo '======================================'
            echo ' CI/CD PIPELINE FAILED'
            echo ' Check the failed stage for details'
            echo '======================================'
        }


        always {

            echo 'Jenkins pipeline execution completed.'
        }
    }
}