pipeline {
    agent any

    environment {
        ACR_NAME = "sowmiyaacr123"
        IMAGE_NAME = "java-app"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        FULL_IMAGE = "${ACR_NAME}.azurecr.io/${IMAGE_NAME}:${IMAGE_TAG}"
        K8S_NAMESPACE = "default"
        RESOURCE_GROUP = "pipelineproject"
        AKS_CLUSTER = "sowmiya-aks"
        TENANT_ID = "98a4779e-cd18-4e1e-9c0f-99a7e7ea9e18"
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

        stage('Login to ACR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'acr-credentials', 
                                                  usernameVariable: 'ACR_USERNAME', 
                                                  passwordVariable: 'ACR_PASSWORD')]) {
                    sh "docker login ${ACR_NAME}.azurecr.io -u $ACR_USERNAME -p $ACR_PASSWORD"
                }
            }
        }

        stage('Push to ACR') {
            steps {
                sh "docker push ${FULL_IMAGE}"
            }
        }

        stage('Deploy to AKS - Initial') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'azure-sp', 
                                                  usernameVariable: 'AZURE_CLIENT_ID', 
                                                  passwordVariable: 'AZURE_CLIENT_SECRET')]) {
                    sh """
                    # Login to Azure
                    az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $TENANT_ID

                    # Get kubeconfig
                    az aks get-credentials --resource-group $RESOURCE_GROUP --name $AKS_CLUSTER --overwrite-existing

                    # Deploy resources
                    kubectl apply -f deployment.yaml --namespace ${K8S_NAMESPACE}
                    kubectl apply -f service.yaml --namespace ${K8S_NAMESPACE}
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
