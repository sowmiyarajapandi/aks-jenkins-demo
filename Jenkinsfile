pipeline {
    agent any

    environment {
        ACR_NAME = "sowmiyaacr123"
        IMAGE_NAME = "java-app"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        FULL_IMAGE = "${ACR_NAME}.azurecr.io/${IMAGE_NAME}:${IMAGE_TAG}"
        K8S_NAMESPACE = "default"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/sowmiyarajapandi/aks-jenkins-demo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${FULL_IMAGE} ."
            }
        }

        stage('Push to ACR') {
            steps {
                sh "az acr login --name ${ACR_NAME}"
                sh "docker push ${FULL_IMAGE}"
            }
        }

        stage('Deploy to AKS') {
            steps {
                sh """
                kubectl set image deployment/java-app-deployment \
                java-app=${FULL_IMAGE} \
                --namespace ${K8S_NAMESPACE}
                """
            }
        }
    }

    post {
        always {
            echo "Cleaning up Docker images on agent..."
            sh 'docker system prune -f'
        }
    }
}