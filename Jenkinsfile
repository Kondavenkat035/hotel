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
                // We ONLY ask Jenkins for the 'k8s' secret
                withCredentials([
                    string(credentialsId: 'k8s', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    script {
                        sh """
                        # PASTE YOUR ACTUAL ACCESS KEY ID HERE (The one starting with AKIA)
                        export AWS_ACCESS_KEY_ID="AKIAxxxxxxxxxxxxxxxx" 
                        
                        # This comes from your 'k8s' Jenkins credential
                        export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                        export AWS_DEFAULT_REGION=${AWS_REGION}

                        # Update hotel.yml with new tag
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
        failure { echo "Deployment Failed. Check AWS Access ID or k8s Secret." }
        always {
            sh "docker rmi ${DOCKER_IMAGE}:${TAG} ${DOCKER_IMAGE}:latest || true"
        }
    }
}
