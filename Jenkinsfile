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
                
                URL=$(kubectl get svc hotel-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
                
                if [ -z "$URL" ]; then
                    URL=$(kubectl get svc hotel-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
                fi

                echo "===================================================="
                if [ -z "$URL" ]; then
                    echo "STATUS: LoadBalancer is still provisioning."
                else
                    echo "Application URL: http://$URL/hotel"
                fi
                echo "===================================================="
                '''
            }
        }
    }

    post {
        success {
            echo "Deployment Successful!"

            emailext (
                subject: "✅ SUCCESS: Jenkins Build #${BUILD_NUMBER}",
                body: """
                <h3>Pipeline Execution Successful</h3>
                <p><b>Job:</b> ${JOB_NAME}</p>
                <p><b>Build Number:</b> ${BUILD_NUMBER}</p>
                <p>Status: SUCCESS ✅</p>
                <p>Check Jenkins for details.</p>
                """,
                to: "kondavenkat035@gmail.com",
                from: "kondavenkat035@gmail.com"
            )
        }

        failure {
            echo "Deployment Failed. Check the logs above for errors."

            emailext (
                subject: "❌ FAILED: Jenkins Build #${BUILD_NUMBER}",
                body: """
                <h3>Pipeline Execution Failed</h3>
                <p><b>Job:</b> ${JOB_NAME}</p>
                <p><b>Build Number:</b> ${BUILD_NUMBER}</p>
                <p>Status: FAILURE ❌</p>
                <p>Please check Jenkins logs.</p>
                """,
                to: "kondavenkat035@gmail.com",
                from: "kondavenkat035@gmail.com"
            )
        }
    }
}
