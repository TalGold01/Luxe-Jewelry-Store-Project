pipeline {
    agent {
        docker {
            image 'talgold01/luxe-jewelry-store-project/jenkins-agent:latest'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    options {
        buildDiscarder(logRotator(daysToKeepStr: '30'))
        disableConcurrentBuilds()
        timestamps()
    }

    environment {
        DOCKER_REGISTRY   = 'talgold01'
        PROJECT_NAME      = 'luxe-jewelry-store-project'

        AUTH_SERVICE_IMAGE = "${DOCKER_REGISTRY}/${PROJECT_NAME}-auth-service:latest"
        BACKEND_IMAGE      = "${DOCKER_REGISTRY}/${PROJECT_NAME}-backend:latest"
        FRONTEND_IMAGE     = "${DOCKER_REGISTRY}/${PROJECT_NAME}-frontend:latest"
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
                    sh "docker build -t ${AUTH_SERVICE_IMAGE} ./auth-service"
                    sh "docker build -t ${BACKEND_IMAGE} ./backend"
                    sh "docker build -t ${FRONTEND_IMAGE} ./frontend"
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
                    sh "snyk test --docker ${BACKEND_IMAGE} --severity-threshold=high"
                    sh "snyk test --docker ${FRONTEND_IMAGE} --severity-threshold=high"
                    sh "snyk test --docker ${AUTH_SERVICE_IMAGE} --severity-threshold=high"
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${AUTH_SERVICE_IMAGE}
                        docker push ${BACKEND_IMAGE}
                        docker push ${FRONTEND_IMAGE}
                    """
                }
            }
        }
    }
}
