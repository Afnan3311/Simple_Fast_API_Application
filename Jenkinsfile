
pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-cred'
        DOCKER_IMAGE          = 'afnan2004/simple_fast_app'

        SONAR_SERVER_NAME     = 'sonarqube_env'
        SONAR_SCANNER_TOOL    = 'sonar-scanner'

        GIT_CREDENTIALS_ID    = 'github-cred'

        MANIFEST_REPO         = 'https://github.com/Afnan3311/Simple_Fast_API_Application.git'
        MANIFEST_BRANCH       = 'main'
    }

    stages {

        // 1. Checkout Source Code
        stage('Checkout Source Code') {
            steps {
                checkout scm
            }
        }

        // 2. SonarQube Analysis
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool "${SONAR_SCANNER_TOOL}"

                    withSonarQubeEnv("${SONAR_SERVER_NAME}") {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=fast-api-chatbot-key \
                            -Dsonar.projectName=fast-api-chatbot-project \
                            -Dsonar.sources=. \
                            -Dsonar.exclusions=**/venv/**,**/.venv/**,**/__pycache__/**,**/.pytest_cache/**,**/static/**,**/manifests/**
                        """
                    }
                }
            }
        }

        // 3. Build and Push Docker Image
        stage('Build & Push Docker Image') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: "${DOCKER_CREDENTIALS_ID}",
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )
                    ]) {
                        sh """
                            echo "\$DOCKER_PASS" | docker login \
                            -u "\$DOCKER_USER" --password-stdin

                            docker build \
                            -t ${DOCKER_IMAGE}:${BUILD_NUMBER} \
                            -t ${DOCKER_IMAGE}:latest \
                            .

                            docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                            docker push ${DOCKER_IMAGE}:latest

                            docker logout
                        """
                    }
                }
            }
        }

        // 4. Update Kubernetes Deployment Manifest
        stage('Update Manifest Repo') {
            steps {
                dir('manifest-workspace') {

                    git branch: "${MANIFEST_BRANCH}",
                        credentialsId: "${GIT_CREDENTIALS_ID}",
                        url: "${MANIFEST_REPO}"

                    withCredentials([
                        usernamePassword(
                            credentialsId: "${GIT_CREDENTIALS_ID}",
                            usernameVariable: 'GH_USER',
                            passwordVariable: 'GH_TOKEN'
                        )
                    ]) {
                        sh '''
                            IMAGE_TAG="${DOCKER_IMAGE}:${BUILD_NUMBER}"

                            sed -i -E \
                            "/name: fastapi-app/{n;s|image:.*|image: ${IMAGE_TAG}|}" \
                            manifests/Deployment.yaml

                            git config user.name "Afnan3311"
                            git config user.email "mohdafnan9617@gmail.com"

                            git add manifests/Deployment.yaml

                            if ! git diff --cached --quiet; then

                                git commit -m "Update FastAPI image to ${BUILD_NUMBER} [skip ci]"

                                PUSH_URL="https://${GH_USER}:${GH_TOKEN}@${MANIFEST_REPO#https://}"

                                git push "$PUSH_URL" HEAD:${MANIFEST_BRANCH}

                            else
                                echo "No manifest changes detected"
                            fi
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully! Docker image pushed and Kubernetes manifest updated.'
        }

        failure {
            echo 'Pipeline failed. Please review the stage logs above.'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}

