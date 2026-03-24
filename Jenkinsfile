pipeline {
    agent any

    environment {
        // Docker Hub Details
        DOCKER_IMAGE = "kondavenkat035/hotal"
        TAG = "${BUILD_NUMBER}"
        
        // AWS/EKS Details
        AWS_REGION = "us-east-1" 
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Pulls your code from GitHub
                git branch: 'main', url: 'https://github.com/Kondavenkat035/hotel.git'
            }
        }

        stage('Maven Build') {
            steps {
                // Compiles the Java project and creates the JAR
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build & Tag') {
            steps {
                // Builds the image and tags it with the Build Number
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
                // This matches your 'k8s' Username/Password credential
                withCredentials([usernamePassword(
                    credentialsId: 'k8s', 
                    usernameVariable: 'MY_AWS_ID', 
                    passwordVariable: 'MY_AWS_SECRET'
                )]) {
                    script {
                        sh """
                        # 1. Set the AWS Identity
                        export AWS_ACCESS_KEY_ID=${MY_AWS_ID}
                        export AWS_SECRET_ACCESS_KEY=${MY_AWS_SECRET}
                        export AWS_DEFAULT_REGION=${AWS_REGION}

                        # 2. Automatically update the hotel.yml image tag
                        # This ensures EKS pulls the NEW version (e.g., :11)
                        sed -i "s|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${TAG}|g" hotel.yml

                        # 3. Apply the changes to the EKS Cluster
                        kubectl apply -f hotel.yml
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
            echo "Deployment Failed. Check logs above for AWS/K8s errors."
        }
        always {
            // Cleanup to keep the Jenkins server clean
            sh "docker rmi ${DOCKER_IMAGE}:${TAG} ${DOCKER_IMAGE}:latest || true"
        }
    }
}
