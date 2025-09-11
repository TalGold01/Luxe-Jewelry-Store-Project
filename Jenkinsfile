pipeline {
    agent {
        docker {
            image 'talgold01/jenkins-agent:latest'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    options {
        buildDiscarder(logRotator(daysToKeepStr: '30'))
        disableConcurrentBuilds()
        timestamps()
    }

    environment {
        DOCKER_REGISTRY = 'talgold01'
        AUTH_SERVICE_IMAGE = "${DOCKER_REGISTRY}/luxe-jewelry-auth-service"
        BACKEND_IMAGE = "${DOCKER_REGISTRY}/luxe-jewelry-backend"
        FRONTEND_IMAGE = "${DOCKER_REGISTRY}/luxe-jewelry-frontend"
        SNYK_TOKEN = credentials('snyk-api-token') // Jenkins secret for Snyk
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Lint & Test') {
            parallel {
                stage('Python Lint') {
                    steps {
                        sh 'pip install -r src/backend/requirements.txt pylint'
                        sh 'python3 -m pylint src/backend/*.py'
                    }
                }

                stage('Frontend Lint') {
                    steps {
                        dir('src/jewelry-store') {
                            sh 'npm install'
                            sh 'npx eslint src/**/*.js'
                        }
                    }
                }

                stage('Unit Tests') {
                    steps {
                        sh 'pip install -r src/backend/requirements.txt pytest'
                        sh 'python3 -m pytest --junitxml=results.xml test/*.py'
                    }
                    post {
                        always {
                            junit allowEmptyResults: true, testResults: 'results.xml'
                        }
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    // Use Git commit hash for image tagging
                    def commitHash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    
                    sh """
                        docker build -t ${AUTH_SERVICE_IMAGE}:${commitHash} src/auth-service
                        docker tag ${AUTH_SERVICE_IMAGE}:${commitHash} ${AUTH_SERVICE_IMAGE}:latest

                        docker build -t ${BACKEND_IMAGE}:${commitHash} src/backend
                        docker tag ${BACKEND_IMAGE}:${commitHash} ${BACKEND_IMAGE}:latest

                        docker build -t ${FRONTEND_IMAGE}:${commitHash} src/jewelry-store
                        docker tag ${FRONTEND_IMAGE}:${commitHash} ${FRONTEND_IMAGE}:latest
                    """
                }
            }
        }

        stage('Snyk Security Scan') {
            steps {
                withEnv(["SNYK_TOKEN=${env.SNYK_TOKEN}"]) {
                    sh """
                        snyk container test ${AUTH_SERVICE_IMAGE}:latest --file=src/auth-service/Dockerfile --severity-threshold=high || true
                        snyk container test ${BACKEND_IMAGE}:latest --file=src/backend/Dockerfile --severity-threshold=high || true
                        snyk container test ${FRONTEND_IMAGE}:latest --file=src/jewelry-store/Dockerfile --severity-threshold=high || true
                    """
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                        docker push ${AUTH_SERVICE_IMAGE}:latest
                        docker push ${BACKEND_IMAGE}:latest
                        docker push ${FRONTEND_IMAGE}:latest
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    docker-compose -f docker-compose.yml up -d --build
                """
            }
        }
    }

    post {
        always {
            sh """
                docker system prune -f
            """
        }
    }
}
