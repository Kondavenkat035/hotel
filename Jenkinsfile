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
                
                # 1. Automatically find the Ingress name using labels
                INGRESS_NAME=$(kubectl get ingress -l app=hotel -o jsonpath='{.items[0].metadata.name}')

                if [ -z "$INGRESS_NAME" ]; then
                    echo "Error: No Ingress found with label app=hotel"
                    exit 1
                fi

                # 2. Get the Hostname or IP from that specific Ingress
                URL=$(kubectl get ingress $INGRESS_NAME -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
                
                # Fallback to IP if hostname is empty
                if [ -z "$URL" ]; then
                    URL=$(kubectl get ingress $INGRESS_NAME -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
                fi

                echo "======================================"
                echo "Detected Ingress: $INGRESS_NAME"
                echo "Application URL:  http://$URL"
                echo "======================================"
                '''
            }
        }
    post {
        success {
            echo "Deployment Successful"
        }
        failure {
            echo "Deployment Failed"
        }
    }
}
