pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub_cred' // Replace with your Jenkins DockerHub credentials ID
        IMAGE_NAME = 'kondavenkat035/hotel:1998'
    }

    stages {
        stage('Checkout') {
            steps {
               git branch: 'main', url: 'https://github.com/Kondavenkat035/hotel.git'
            }
        }

        stage('Build WAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Push Docker Image') {
            steps {
               usernamePassword(
                credentialsId: 'valaxy-docker',
                usernameVariable: 'DOCKER_USERNAME',
                passwordVariable: 'DOCKER_PASSWORD'
                 )
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${IMAGE_NAME}:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    docker stop restaurant || true
                    docker rm restaurant || true
                    docker run -d -p 4444:8080 --name restaurant ${IMAGE_NAME}:latest
                '''
            }
        }
    }
}

