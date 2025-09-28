@Library('luxe-project-shared-lib') _

pipeline {
    agent {
        docker {
            image 'talgold01/luxe-jewelry-store-project:jenkins-agent'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
            reuseNode true
            registryCredentialsId 'docker-hub'
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
        DEPLOY_ENV          = 'development' // change to staging or production as needed

        // Docker Hub tags (single repo with different tags)
        JENKINS_AGENT_IMAGE  = "${DOCKERHUB_REGISTRY}/${PROJECT_NAME}:jenkins-agent"
        AUTH_SERVICE_IMAGE   = "${DOCKERHUB_REGISTRY}/${PROJECT_NAME}:auth-service"
        BACKEND_IMAGE        = "${DOCKERHUB_REGISTRY}/${PROJECT_NAME}:backend"
        FRONTEND_IMAGE       = "${DOCKERHUB_REGISTRY}/${PROJECT_NAME}:frontend"

        // Nexus tags
        JENKINS_AGENT_IMAGE_NX  = "${NEXUS_REGISTRY}/jenkins-agent"
        AUTH_SERVICE_IMAGE_NX   = "${NEXUS_REGISTRY}/auth-service"
        BACKEND_IMAGE_NX        = "${NEXUS_REGISTRY}/backend"
        FRONTEND_IMAGE_NX       = "${NEXUS_REGISTRY}/frontend"
        GITHUB_CREDENTIALS       = credentials('github-creds')
        SNYK_TOKEN               = credentials('snyk-token')

    }

    triggers {
        githubPush()
    }

    stages {
        stage('Setup Docker Compose') {
            steps {
                script {
                    echo "Installing Docker Compose..."
                    sh 'apt-get update && apt-get install -y docker-compose'
                    sh 'cp infra/docker-compose.yml /tmp/docker-compose.yml'
                }
            }
        }
        
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/TalGold01/Luxe-Jewelry-Store-Project.git',
                    credentialsId: 'github-creds'
            }
        }
        // ... (rest of your stages remain the same)
        stage('Build Docker Images') {
            steps {
                script {
                    dockerUtils.buildAndTagImage('jenkins-agent', JENKINS_AGENT_IMAGE, JENKINS_AGENT_IMAGE_NX)
                    dockerUtils.buildAndTagImage('auth-service', AUTH_SERVICE_IMAGE, AUTH_SERVICE_IMAGE_NX)
                    dockerUtils.buildAndTagImage('backend', BACKEND_IMAGE, BACKEND_IMAGE_NX)
                    dockerUtils.buildAndTagImage('frontend', FRONTEND_IMAGE, FRONTEND_IMAGE_NX)
                }
            }
        }

        stage('Pull Latest Images') {
            steps {
                script {
                    dockerUtils.pullLatestImages(JENKINS_AGENT_IMAGE, AUTH_SERVICE_IMAGE, BACKEND_IMAGE, FRONTEND_IMAGE)
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
                script {
                    withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
                        sh "snyk auth ${SNYK_TOKEN}"
                        dockerUtils.snykScan(JENKINS_AGENT_IMAGE)
                        dockerUtils.snykScan(AUTH_SERVICE_IMAGE)
                        dockerUtils.snykScan(BACKEND_IMAGE)
                        dockerUtils.snykScan(FRONTEND_IMAGE)
                    }
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    // Docker Hub
                    dockerUtils.pushToDockerHub(JENKINS_AGENT_IMAGE)
                    dockerUtils.pushToDockerHub(AUTH_SERVICE_IMAGE)
                    dockerUtils.pushToDockerHub(BACKEND_IMAGE)
                    dockerUtils.pushToDockerHub(FRONTEND_IMAGE)

                    // Nexus
                    dockerUtils.pushToNexus(JENKINS_AGENT_IMAGE_NX, NEXUS_REGISTRY)
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

                    dir('infra') {
                        // Stop existing containers if running
                        sh 'docker-compose down || true'

                        // Deploy new containers
                        sh "docker-compose -f docker-compose.yml up -d"

                        echo "Deployment complete for ${DEPLOY_ENV}"
                    }
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
