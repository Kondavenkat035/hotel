pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "kondavenkat035/hotal"
        TAG = "${BUILD_NUMBER}"
        AWS_REGION = "us-east-1"
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Kondavenkat035/hotel.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build & Tag') {
            steps {
                sh """
                docker build -t ${DOCKER_IMAGE}:${TAG} .
                docker tag ${DOCKER_IMAGE}:${TAG} ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'valaxy-docker',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push ${DOCKER_IMAGE}:${TAG}
                    docker push ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // FIXED: This matches your 'Username with password' credential named 'k8s'
                withCredentials([usernamePassword(
                    credentialsId: 'k8s', 
                    usernameVariable: 'MY_AWS_ID', 
                    passwordVariable: 'MY_AWS_SECRET'
                )]) {
                    script {
                        sh """
                        # We map the credentials to the standard AWS variables
                        export AWS_ACCESS_KEY_ID=${MY_AWS_ID}
                        export AWS_SECRET_ACCESS_KEY=${MY_AWS_SECRET}
                        export AWS_DEFAULT_REGION=${AWS_REGION}

                        # Update the hotel.yml image tag to the current Build Number
                        sed -i "s|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${TAG}|g" hotel.yml

                        # Apply to EKS
                        kubectl apply -f hotel.yml
                        """
                    }
                }
            }
        }
    }

    post {
        success { echo "Deployment Successful!" }
        failure { echo "Deployment Failed. Check logs above." }
        always {
            sh "docker rmi ${DOCKER_IMAGE}:${TAG} ${DOCKER_IMAGE}:latest || true"
        }
    }
}
