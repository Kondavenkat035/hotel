pipeline {
    agent any

    environment {
        // Docker Hub Details
        DOCKER_IMAGE = "kondavenkat035/hotal"
        TAG = "${BUILD_NUMBER}"
        
        // AWS and Kubernetes Details
        AWS_REGION = "us-east-1"
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Pulls your code from the GitHub repository
                git branch: 'main', url: 'https://github.com/Kondavenkat035/hotel.git'
            }
        }

        stage('Maven Build') {
            steps {
                // Compiles your Java application and skips unit tests for speed
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build & Tag') {
            steps {
                // Builds the Docker image and tags it with the Jenkins Build Number
                sh """
                docker build -t ${DOCKER_IMAGE}:${TAG} .
                docker tag ${DOCKER_IMAGE}:${TAG} ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Push to DockerHub') {
            steps {
                // Uses your 'valaxy-docker' credentials to log in to Docker Hub
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
                // This 'withCredentials' block provides the "ID Card" (AWS Keys) for EKS
                // It uses your 'k8s' credential (Username/Password type)
                withCredentials([usernamePassword(
                    credentialsId: 'k8s', 
                    usernameVariable: 'AWS_ID', 
                    passwordVariable: 'AWS_SECRET'
                )]) {
                    script {
                        sh """
                        # 1. Provide the credentials to the environment for AWS CLI
                        export AWS_ACCESS_KEY_ID=${AWS_ID}
                        export AWS_SECRET_ACCESS_KEY=${AWS_SECRET}
                        export AWS_DEFAULT_REGION=${AWS_REGION}
                        export KUBECONFIG=${KUBECONFIG}

                        # 2. Update the hotel.yml image tag to the current Build Number
                        # This tells Kubernetes to pull the NEW version of the image
                        sed -i "s|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${TAG}|g" hotel.yml

                        # 3. Apply the configuration to the EKS Cluster
                        kubectl apply -f hotel.yml
                        
                        # 4. Optional: If you have other files like auto.yml or food.yml, add them here
                        # kubectl apply -f auto.yml
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Successfully deployed Build #${env.BUILD_NUMBER} to Kubernetes!"
        }
        failure {
            echo "Deployment Failed. Check the logs for AWS Authentication or K8s connection errors."
        }
        always {
            // Removes local images from the Jenkins server to save disk space
            sh "docker rmi ${DOCKER_IMAGE}:${TAG} ${DOCKER_IMAGE}:latest || true"
        }
    }
}
