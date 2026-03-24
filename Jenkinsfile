pipeline {
    agent any

    environment {
        // Docker Details
        DOCKER_IMAGE = "kondavenkat035/hotal"
        TAG = "${BUILD_NUMBER}"
        
        // AWS Details (Change region if yours is different)
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
                // Compiles your Java code and creates the JAR file
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

        stage('Deploy to Kubernetes (EKS)') {
            steps {
                // This block injects your AWS "ID Card" so kubectl can talk to EKS
                withCredentials([
                    string(credentialsId: 'k8s', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'AWS_SECRET_KEY', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    script {
                        sh """
                        # 1. Set AWS Credentials in the environment
                        export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
                        export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                        export AWS_DEFAULT_REGION=${AWS_REGION}

                        # 2. Update the hotel.yml with the new image tag (Build Number)
                        # This ensures K8s pulls the NEW image, not the old 'latest'
                        sed -i 's|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${TAG}|g' hotel.yml

                        # 3. Apply the changes to the cluster
                        kubectl apply -f hotel.yml
                        
                        # 4. Optional: Verify the deployment status
                        # Replace 'hotel-deployment' with the actual name in your hotel.yml
                        # kubectl rollout status deployment/hotel-deployment
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Successfully deployed Build #${env.BUILD_NUMBER} to EKS!"
        }
        failure {
            echo "Deployment failed. Check the logs above for AWS or K8s errors."
        }
        always {
            // Clean up Docker images on the Jenkins server to save disk space
            sh "docker rmi ${DOCKER_IMAGE}:${TAG} ${DOCKER_IMAGE}:latest || true"
        }
    }
}
