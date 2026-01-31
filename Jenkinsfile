pipeline {
    agent any

    parameters {
        choice(
            name: 'ACTION',
            choices: ['build', 'deploy', 'remove'],
            description: 'Choose pipeline action'
        )

        string(name: 'IMAGE_NAME', defaultValue: 'spring_project2003', description: 'Docker image name')
        string(name: 'IMAGE_TAG', defaultValue: 'v1', description: 'Docker image tag')
        string(name: 'DOCKERHUB_USERNAME', defaultValue: 'nisargasj2907', description: 'DockerHub username')
    }

    environment {
        IMAGE = "${params.DOCKERHUB_USERNAME}/${params.IMAGE_NAME}:${params.IMAGE_TAG}"
        CONTAINER_NAME = "spring_project_container"
    }

    stages {

        stage('Checkout Code') {
            when {
                expression { params.ACTION == 'build' }
            }
            steps {
                echo "Checking out source code"
                git 'https://github.com/nisarga2907/spring_project2003.git'
            }
        }

        stage('Build Docker Image') {
            when {
                expression { params.ACTION == 'build' }
            }
            steps {
                echo "Building Docker image: ${IMAGE}"
                sh 'docker build -t $IMAGE .'
            }
        }

        stage('Docker Login') {
            when {
                expression { params.ACTION == 'build' }
            }
            steps {
                echo "Logging in to Docker Hub"
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-3153',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Docker Push') {
            when {
                expression { params.ACTION == 'build' }
            }
            steps {
                echo "Pushing image to Docker Hub"
                sh 'docker push $IMAGE'
            }
        }

        stage('Delete Local Image') {
            when {
                expression { params.ACTION == 'build' }
            }
            steps {
                echo "Deleting local Docker image"
                sh 'docker rmi $IMAGE || true'
            }
        }

        stage('Deploy') {
            when {
                expression { params.ACTION == 'deploy' }
            }
            steps {
                echo "Deploying application"
                sh '''
                docker-compose down || true
                docker-compose up -d
                '''
            }
        }

        stage('Remove') {
            when {
                expression { params.ACTION == 'remove' }
            }
            steps {
                echo "Removing container and image"
                sh '''
                docker rm -f spring_project_container || true
                docker rmi -f $IMAGE || true
                '''
            }
        }
    }

    post {
        always {
            echo "Logging out from Docker Hub"
            sh 'docker logout || true'
            echo "Pipeline execution completed"
        }
    }
}
