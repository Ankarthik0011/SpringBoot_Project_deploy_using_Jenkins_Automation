pipeline {
    agent any

    parameters {
        choice(
            name: 'ACTION',
            choices: ['IMAGE_FLOW', 'COMPOSE_FLOW'],
            description: 'Choose Docker Image flow or Docker Compose flow'
        )
    }

    environment {
        IMAGE_NAME   = "randomuser/springboot-app"
        IMAGE_TAG    = "v1"
        DOCKER_CREDS = "dockerhub-3153"
    }

    stages {

        /* ---------- IMAGE FLOW ---------- */
        stage('Build Maven & Docker Image') {
            when {
                expression { params.ACTION == 'IMAGE_FLOW' }
            }
            steps {
                sh '''
                  echo "Building Maven project..."
                  mvn clean package

                  echo "Building Docker image..."
                  docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Docker Login') {
            when {
                expression { params.ACTION == 'IMAGE_FLOW' }
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDS}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Image to DockerHub') {
            when {
                expression { params.ACTION == 'IMAGE_FLOW' }
            }
            steps {
                sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Remove Local Docker Image') {
            when {
                expression { params.ACTION == 'IMAGE_FLOW' }
            }
            steps {
                sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
            }
        }

        stage('Docker Logout') {
            when {
                expression { params.ACTION == 'IMAGE_FLOW' }
            }
            steps {
                sh 'docker logout'
            }
        }

        /* ---------- COMPOSE FLOW ---------- */
        stage('Docker Compose Run') {
            when {
                expression { params.ACTION == 'COMPOSE_FLOW' }
            }
            steps {
                sh 'docker-compose up -d'
            }
        }

        stage('Docker Compose Remove') {
            when {
                expression { params.ACTION == 'COMPOSE_FLOW' }
            }
            steps {
                sh 'docker-compose down'
            }
        }
    }

    /* ---------- POST ACTIONS ---------- */
    post {
        success {
            echo '✅ Pipeline executed successfully'
        }
        failure {
            echo '❌ Pipeline failed'
        }
        always {
            echo '🧹 Post cleanup completed'
        }
    }
}
