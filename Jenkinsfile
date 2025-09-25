@Library('luxe-project-shared-lib') _

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
        
        DEPLOY_ENV          = 'development' // can be changed to 'staging' or 'production'

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
                    dockerUtils.buildAndTagImage('auth-service', AUTH_SERVICE_IMAGE, AUTH_SERVICE_IMAGE_NX)
                    dockerUtils.buildAndTagImage('backend', BACKEND_IMAGE, BACKEND_IMAGE_NX)
                    dockerUtils.buildAndTagImage('frontend', FRONTEND_IMAGE, FRONTEND_IMAGE_NX)
                }
            }
        }

        stage('Pull Docker Images') {
            steps {
                script {
                    dockerUtils.pullLatestImages(AUTH_SERVICE_IMAGE, BACKEND_IMAGE, FRONTEND_IMAGE)
                }
            }
        }

        stage('Unit Tests') {
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
                    dockerUtils.snykScan(AUTH_SERVICE_IMAGE)
                    dockerUtils.snykScan(BACKEND_IMAGE)
                    dockerUtils.snykScan(FRONTEND_IMAGE)
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    dockerUtils.pushToDockerHub(AUTH_SERVICE_IMAGE)
                    dockerUtils.pushToDockerHub(BACKEND_IMAGE)
                    dockerUtils.pushToDockerHub(FRONTEND_IMAGE)

                    dockerUtils.pushToNexus(AUTH_SERVICE_IMAGE_NX, NEXUS_REGISTRY)
                    dockerUtils.pushToNexus(BACKEND_IMAGE_NX, NEXUS_REGISTRY)
                    dockerUtils.pushToNexus(FRONTEND_IMAGE_NX, NEXUS_REGISTRY)
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo "Deploying services to environment: ${DEPLOY_ENV}"

                    // Optional: stop existing containers before deploy
                    sh 'docker-compose down || true'

                    // Run containers using latest images
                    sh "docker-compose -f docker-compose.yml up -d"

                    echo "Deployment complete for ${DEPLOY_ENV}"
                }
            }
        }
    }

    post {
        always {
            script {
                echo "Cleaning up Docker images from Jenkins agent"
                sh "docker system prune -af || true"
            }
        }
    }
}
