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

        AUTH_SERVICE_IMAGE_DH = "${DOCKERHUB_REGISTRY}/${PROJECT_NAME}-auth-service:latest"
        BACKEND_IMAGE_DH      = "${DOCKERHUB_REGISTRY}/${PROJECT_NAME}-backend:latest"
        FRONTEND_IMAGE_DH     = "${DOCKERHUB_REGISTRY}/${PROJECT_NAME}-frontend:latest"

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
                    sh "docker build -t ${AUTH_SERVICE_IMAGE_DH} ./auth-service"
                    sh "docker build -t ${BACKEND_IMAGE_DH} ./backend"
                    sh "docker build -t ${FRONTEND_IMAGE_DH} ./frontend"
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    dir('/projects/Luxe-Jewelry-Store-Project/test') {
                        sh 'pytest --maxfail=1 --disable-warnings -q'
                    }
                }
            }
        }

        stage('Snyk Scan') {
            steps {
                withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
                    sh "snyk auth ${SNYK_TOKEN}"
                    sh "snyk test --docker ${BACKEND_IMAGE_DH} --severity-threshold=high"
                    sh "snyk test --docker ${FRONTEND_IMAGE_DH} --severity-threshold=high"
                    sh "snyk test --docker ${AUTH_SERVICE_IMAGE_DH} --severity-threshold=high"
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                // Push to DockerHub
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${AUTH_SERVICE_IMAGE_DH}
                        docker push ${BACKEND_IMAGE_DH}
                        docker push ${FRONTEND_IMAGE_DH}
                    """
                }

                // Tag & Push to Nexus
                withCredentials([usernamePassword(credentialsId: 'nexus-creds', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sh """
                        echo "$NEXUS_PASS" | docker login -u "$NEXUS_USER" --password-stdin $NEXUS_REGISTRY
                        docker tag ${AUTH_SERVICE_IMAGE_DH} ${AUTH_SERVICE_IMAGE_NX}
                        docker tag ${BACKEND_IMAGE_DH} ${BACKEND_IMAGE_NX}
                        docker tag ${FRONTEND_IMAGE_DH} ${FRONTEND_IMAGE_NX}
                        docker push ${AUTH_SERVICE_IMAGE_NX}
                        docker push ${BACKEND_IMAGE_NX}
                        docker push ${FRONTEND_IMAGE_NX}
                    """
                }
            }
        }
    }
}
