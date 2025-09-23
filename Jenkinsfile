pipeline {
    agent {
        docker {
            image 'talgold01/luxe-jewelry-store-project:jenkins-agent-latest'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
            reuseNode true
        }
    }

    options {
        buildDiscarder(logRotator(daysToKeepStr: '30'))
        disableConcurrentBuilds()
        timestamps()
    }

    environment {
        DOCKERHUB_REGISTRY  = 'talgold01'
        NEXUS_REGISTRY      = 'localhost:8082/docker-hosted'
        PROJECT_NAME        = 'luxe-jewelry-store-project'

        AUTH_SERVICE_IMAGE   = "${DOCKERHUB_REGISTRY}/${PROJECT_NAME}-auth-service:latest"
        BACKEND_IMAGE        = "${DOCKERHUB_REGISTRY}/${PROJECT_NAME}-backend:latest"
        FRONTEND_IMAGE       = "${DOCKERHUB_REGISTRY}/${PROJECT_NAME}-frontend:latest"

        AUTH_SERVICE_IMAGE_NX = "${NEXUS_REGISTRY}/auth-service:latest"
        BACKEND_IMAGE_NX      = "${NEXUS_REGISTRY}/backend:latest"
        FRONTEND_IMAGE_NX     = "${NEXUS_REGISTRY}/frontend:latest"
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/TalGold01/Luxe-Jewelry-Store-Project.git',
                    credentialsId: 'github-creds'
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    // Build all images from docker-compose.yml
                    sh 'docker-compose build'
                }
            }
        }

        stage('Pull Docker Images') {
            steps {
                script {
                    // Pull latest images from DockerHub to compare; don't fail if missing
                    sh "docker pull ${AUTH_SERVICE_IMAGE} || true"
                    sh "docker pull ${BACKEND_IMAGE} || true"
                    sh "docker pull ${FRONTEND_IMAGE} || true"
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    dir('test') {
                        sh 'pytest --maxfail=1 --disable-warnings -q'
                    }
                }
            }
        }

        stage('Snyk Scan') {
            steps {
                withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
                    sh "snyk auth ${SNYK_TOKEN}"
                    sh "snyk test --docker ${BACKEND_IMAGE} --severity-threshold=high"
                    sh "snyk test --docker ${FRONTEND_IMAGE} --severity-threshold=high"
                    sh "snyk test --docker ${AUTH_SERVICE_IMAGE} --severity-threshold=high"
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    // Push to DockerHub using docker-compose
                    withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker-compose push
                        """
                    }

                    // Tag & Push to Nexus
                    withCredentials([usernamePassword(credentialsId: 'nexus-creds', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                        sh """
                            echo "$NEXUS_PASS" | docker login -u "$NEXUS_USER" --password-stdin $NEXUS_REGISTRY
                            docker tag ${AUTH_SERVICE_IMAGE} ${AUTH_SERVICE_IMAGE_NX}
                            docker tag ${BACKEND_IMAGE} ${BACKEND_IMAGE_NX}
                            docker tag ${FRONTEND_IMAGE} ${FRONTEND_IMAGE_NX}
                            docker push ${AUTH_SERVICE_IMAGE_NX}
                            docker push ${BACKEND_IMAGE_NX}
                            docker push ${FRONTEND_IMAGE_NX}
                        """
                    }
                }
            }
        }
    }
}
