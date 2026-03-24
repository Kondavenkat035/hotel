pipeline {
    agent any

    environment {
        // Docker Hub Details
        DOCKER_IMAGE = "kondavenkat035/hotal"
        TAG = "${BUILD_NUMBER}"
        
        // AWS and EKS Details
        AWS_REGION = "us-east-1"
        CLUSTER_NAME = "new-cluster"
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

        stage('Deploy to EKS') {
            steps {
                // This uses your 'k8s' Username/Password credential for AWS Access
                withCredentials([usernamePassword(
                    credentialsId: 'k8s', 
                    usernameVariable: 'AWS_ID', 
                    passwordVariable: 'AWS_SECRET'
                )]) {
                    script {
                        sh """
                        # 1. Set AWS credentials for the session
                        export AWS_ACCESS_KEY_ID='${AWS_ID}'
                        export AWS_SECRET_ACCESS_KEY='${AWS_SECRET}'
                        export AWS_DEFAULT_REGION=${AWS_REGION}
                        export KUBECONFIG=${KUBECONFIG}

                        # 2. Update the Kubeconfig for your specific cluster
                        # This generates the token that fixes the "provide credentials" error
                        aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}

                        # 3. Update the image tag in hotel.yml
                        sed -i "s|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${TAG}|g" hotel.yml

                        # 4. Apply the configuration to the cluster
                        kubectl apply -f hotel.yml
                        
                        # 5. Optional: Apply other files if they exist
                        # kubectl apply -f auto.yml
                        # kubectl apply -f insta.yml
                        # kubectl apply -f takeit.yml
                        # kubectl apply -f food.yml
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Successfully deployed Build #${env.BUILD_NUMBER} to ${CLUSTER_NAME}!"
        }
        failure {
            echo "Deployment Failed. Check the logs for AWS Authentication or Cluster Name errors."
        }
        always {
            // Cleanup local Docker images to save disk space
            sh "docker rmi ${DOCKER_IMAGE}:${TAG} ${DOCKER_IMAGE}:latest || true"
        }
    }
}
