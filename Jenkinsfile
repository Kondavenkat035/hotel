pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "kondavenkat035/hotal"
        TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Kondavenkat035/hotel.git'
            }
        }

        stage('Install') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${TAG} ."
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'valaxy-docker',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                sh """
                docker push ${DOCKER_IMAGE}:${TAG}
                docker tag ${DOCKER_IMAGE}:${TAG} ${DOCKER_IMAGE}:latest
                docker push ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                export KUBECONFIG=/var/lib/jenkins/.kube/config
                kubectl apply -f hotel.yml
                """
            }
        }

        stage('Show App URL') {
            steps {
                sh '''
                export KUBECONFIG=/var/lib/jenkins/.kube/config
                
                # Try to get the Hostname (AWS/Azure)
                URL=$(kubectl get svc hotel-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
                
                # If Hostname is empty, try to get the IP (GCP/Bare Metal)
                if [ -z "$URL" ]; then
                    URL=$(kubectl get svc hotel-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
                fi

                # Display the URL or a helpful message if still pending
                echo "===================================================="
                if [ -z "$URL" ]; then
                    echo "STATUS: Deployment complete, but LoadBalancer is still provisioning."
                    echo "Run 'kubectl get svc hotel-service' in a few moments to get the IP."
                else
                    echo "Application is LIVE at the following links:"
                    echo "hotel:   http://$URL/hotel"
                fi
                echo "===================================================="
                '''
            }
        }
    }

    post {
        success {
            echo "Deployment Successful!"
        }
        failure {
            echo "Deployment Failed. Check the logs above for errors."
        }
    }
}
