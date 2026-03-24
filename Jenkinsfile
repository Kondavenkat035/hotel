pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "kondavenkat035/hotal"
        TAG = "${BUILD_NUMBER}"
        // Set KUBECONFIG globally so every 'sh' step sees it
        KUBECONFIG = "/var/lib/jenkins/.kube/config" 
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Kondavenkat035/hotel.git'
            }
        }

        stage('Install') { // Fixed typo "Inntall"
            steps {
                // Ensure Maven is installed on your Jenkins agent
                sh 'mvn clean package'
            }
        }

        stage('Build & Tag Docker Image') {
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
                script {
                    // Update the YAML file with the new image tag before applying
                    // This ensures K8s actually pulls the NEW version
                    sh "sed -i 's|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${TAG}|g' hotel.yml"
                    
                    sh "kubectl apply -f hotel.yml"
                    
                    // Verify the rollout
                    sh "kubectl rollout status deployment/your-deployment-name"
                }
            }
        }
    }

    post {
        success { echo "Deployment Successful" }
        failure { echo "Deployment Failed" }
        always {
            // Clean up to save disk space
            sh "docker rmi ${DOCKER_IMAGE}:${TAG} ${DOCKER_IMAGE}:latest || true"
        }
    }
}
