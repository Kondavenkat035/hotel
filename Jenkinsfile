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
                withCredentials([usernamePassword(
                    credentialsId: 'k8s', 
                    usernameVariable: 'AWS_ID', 
                    passwordVariable: 'AWS_SECRET'
                )]) {
                    script {
                        sh """
                        # 1. Ensure Jenkins can find AWS and Kubectl in the system path
                        export PATH=\$PATH:/usr/local/bin:/usr/bin:/bin
                        
                        # 2. Set AWS credentials for the handshake
                        export AWS_ACCESS_KEY_ID='${AWS_ID}'
                        export AWS_SECRET_ACCESS_KEY='${AWS_SECRET}'
                        export AWS_DEFAULT_REGION=${AWS_REGION}
                        export KUBECONFIG=${KUBECONFIG}

                        # 3. Refresh the EKS token for 'new-cluster'
                        aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}

                        # 4. Update the image tag in hotel.yml
                        sed -i "s|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${TAG}|g" hotel.yml

                        # 5. Deploy to the cluster (using --validate=false to bypass the credential prompt)
                        kubectl apply -f hotel.yml --validate=false
                        
                        # Apply your other service files if they are in your repo
                        # kubectl apply -f auto.yml --validate=false
                        # kubectl apply -f insta.yml --validate=false
                        # kubectl apply -f takeit.yml --validate=false
                        # kubectl apply -f food.yml --validate=false
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "=========================================================="
            echo "Successfully deployed Build #${env.BUILD_NUMBER} to EKS!"
            echo "=========================================================="
        }
        failure {
            echo "Deployment Failed. Check AWS CLI version or Cluster Name."
        }
        always {
            // Cleanup local images to keep the Jenkins server clean
            sh "docker rmi ${DOCKER_IMAGE}:${TAG} ${DOCKER_IMAGE}:latest || true"
        }
    }
}
