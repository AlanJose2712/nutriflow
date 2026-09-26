pipeline {
    agent any

environment {
    BACKEND_IMAGE = "nutriflow-backend:latest"
    FRONTEND_IMAGE = "nutriflow-frontend:latest"
    MINIKUBE_HOME = "/home/ubuntu/.minikube"
}

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Frontend Build') {
            steps {
                dir('frontend') {
                    sh 'npm ci'
                    sh 'npm run build'
                }
            }
        }

        stage('Backend Dependencies') {
            steps {
                dir('backend') {
                    sh 'npm ci --omit=dev'
                }
            }
        }

stage('SonarQube Analysis') {
    steps {
        script {
            echo "Running SonarQube analysis..."

            def scannerHome = tool 'sonarscanner'

            withSonarQubeEnv('sonarqube') {
                sh """
                    ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.projectName="${SONAR_PROJECT_NAME}" \
                        -Dsonar.sources=frontend/src,backend \
                        -Dsonar.exclusions="**/node_modules/**,**/dist/**,**/build/**,**/coverage/**"
                """
            }
        }
    }
}

        stage('Build Backend Docker Image') {
            steps {
                sh '''
                    docker build \
                      -f Dockerfile.backend \
                      -t ${BACKEND_IMAGE} .
                '''
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                sh '''
                    docker build \
                      --build-arg VITE_API_URL=/nutriflow \
                      -f Dockerfile.frontend \
                      -t ${FRONTEND_IMAGE} .
                '''
            }
        }

        stage('Load Images into Minikube') {
            steps {
                sh '''
                    minikube image load ${BACKEND_IMAGE}
                    minikube image load ${FRONTEND_IMAGE}
                '''
            }
        }

        stage('Validate Kubernetes') {
            steps {
                sh '''
                    kubectl -n nutriflow get deployments
                    kubectl -n nutriflow get pods
                    kubectl -n nutriflow get services
                    kubectl -n nutriflow get ingress
                '''
            }
        }
    }

    post {
        success {
            echo 'NutriFlow Jenkins pipeline completed successfully.'
        }

        failure {
            echo 'NutriFlow Jenkins pipeline failed.'
        }
    }
}
