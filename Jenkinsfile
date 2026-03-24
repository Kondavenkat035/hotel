pipeline {
    agent any

    environment {
        // Docker Hub Details
        DOCKER_IMAGE = "kondavenkat035/hotal"
        TAG = "${BUILD_NUMBER}"
        
        // AWS and EKS Details
        AWS_REGION = "us-east-1"
        CLUSTER_NAME = "new-cluster"
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
                        # 1. Force the use of the new AWS CLI Version 2 path
                        export PATH=/usr/local/bin:/usr/bin:/bin:\$PATH
                        
                        # 2. Setup AWS Environment Credentials
                        export AWS_ACCESS_KEY_ID='${AWS_ID}'
                        export AWS_SECRET_ACCESS_KEY='${AWS_SECRET}'
                        export AWS_DEFAULT_REGION=${AWS_REGION}
                        
                        # 3. Create a fresh Kubeconfig in the current workspace
                        # This avoids permission issues with /var/lib/jenkins/.kube/
                        export KUBECONFIG=\$(pwd)/kubeconfig
                        
                        # 4. Refresh connection using AWS CLI v2
                        aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}

                        # 5. Update image tag in hotel.yml
                        sed -i "s|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${TAG}|g" hotel.yml

                        # 6. Deploy (Bypassing validation to prevent OpenAPI credential errors)
                        kubectl apply -f hotel.yml --validate=false
                        
                        # Optional: Apply other files if they are in your repository
                        # kubectl apply -f auto.yml --validate=false
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
            echo "SUCCESS: Build #${env.BUILD_NUMBER} is live on EKS!"
            echo "=========================================================="
        }
        failure {
            echo "FAILURE: Deployment failed. Verify AWS CLI v2 is in /usr/local/bin"
        }
        always {
            // Cleanup local images to save disk space on Jenkins instance
            sh "docker rmi ${DOCKER_IMAGE}:${TAG} ${DOCKER_IMAGE}:latest || true"
        }
    }
}
