pipeline {
    agent any

    environment {
        DOCKER_USER = 'vijaygandla'
    }

    stages {
        stage('Phase 1: Backend Tests & Build') {
            steps {
                echo 'Running Spring Boot Microservices Tests & Build...'
                dir('backend') {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Phase 2: SonarQube Quality Scan') {
            steps {
                echo 'Sending backend code to SonarQube...'
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    dir('backend') {
                        sh """
                        mvn sonar:sonar \
                          -Dsonar.projectKey=revpay-backend \
                          -Dsonar.host.url=http://sonarqube:9000 \
                          -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Phase 3: Frontend Tests (Angular)') {
            steps {
                echo 'Running Angular Headless Tests...'
                dir('frontend') {
                    sh 'npm install'
                    sh 'npx ng test --watch=false --browsers=ChromeHeadlessNoSandbox'
                }
            }
        }

        stage('Phase 4: Docker Hub Release') {
            steps {
                echo 'Building and Pushing Docker Images...'
                withCredentials([string(credentialsId: 'docker-hub-credentials', variable: 'DOCKER_PASSWORD')]) {
                    sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASSWORD}"

                    // Infrastructure Components
                    dir('backend') {
                        echo 'Building Infrastructure Images...'
                        sh "docker build -t ${DOCKER_USER}/revpay-discovery-server:latest ./infrastructure/discovery-server"
                        sh "docker push ${DOCKER_USER}/revpay-discovery-server:latest"

                        sh "docker build -t ${DOCKER_USER}/revpay-config-server:latest ./infrastructure/config-server"
                        sh "docker push ${DOCKER_USER}/revpay-config-server:latest"

                        sh "docker build -t ${DOCKER_USER}/revpay-api-gateway:latest ./infrastructure/api-gateway"
                        sh "docker push ${DOCKER_USER}/revpay-api-gateway:latest"

                        echo 'Building Microservices Images...'
                        sh "docker build -t ${DOCKER_USER}/revpay-user-service:latest ./microservices/user-service"
                        sh "docker push ${DOCKER_USER}/revpay-user-service:latest"

                        sh "docker build -t ${DOCKER_USER}/revpay-wallet-service:latest ./microservices/wallet-service"
                        sh "docker push ${DOCKER_USER}/revpay-wallet-service:latest"

                        sh "docker build -t ${DOCKER_USER}/revpay-transaction-service:latest ./microservices/transaction-service"
                        sh "docker push ${DOCKER_USER}/revpay-transaction-service:latest"

                        sh "docker build -t ${DOCKER_USER}/revpay-invoice-service:latest ./microservices/invoice-service"
                        sh "docker push ${DOCKER_USER}/revpay-invoice-service:latest"

                        sh "docker build -t ${DOCKER_USER}/revpay-loan-service:latest ./microservices/loan-service"
                        sh "docker push ${DOCKER_USER}/revpay-loan-service:latest"

                        sh "docker build -t ${DOCKER_USER}/revpay-notification-service:latest ./microservices/notification-service"
                        sh "docker push ${DOCKER_USER}/revpay-notification-service:latest"
                    }

                    // Frontend UI
                    dir('frontend') {
                        echo 'Building Frontend Image...'
                        sh "docker build -t ${DOCKER_USER}/revpay-frontend:latest ."
                        sh "docker push ${DOCKER_USER}/revpay-frontend:latest"
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'PIPELINE SUCCESS! All components tested, scanned, and published.'
        }
        failure {
            echo 'PIPELINE FAILED! Check the Jenkins logs.'
        }
    }
}
