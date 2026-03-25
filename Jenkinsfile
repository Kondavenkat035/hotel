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
        stage('Inntall') {
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
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
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
    }

       post {
          success {
            echo "Deployment Successful "
          }
         failure {
            echo "Deployment Failed "
         }
      }
 }
 stage('Show App URL') {
            steps {
        sh '''
            export KUBECONFIG=/var/lib/jenkins/.kube/config

            URL=$(kubectl get ingress k8s-ingress -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')

            echo "======================================"
            echo "Application deployed successfully"
            echo ""
            echo "Food:   http://$URL/food"
            echo "Travel: http://$URL/travel"
            echo "DevOps: http://$URL/devops"
            
            echo ""
            echo "======================================"
        '''
            }
       }
    }
}

     


