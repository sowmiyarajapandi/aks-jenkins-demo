pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'rajcosivadevops/cicdncplimage'
        DOCKER_TAG = 'latest'
    }

    stages {
       
       
        stage('Docker Login & Pull') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhublogin', 'passwordVariable': 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    }
                    sh "docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}"
                }
            }
        }
     
      

        stage('Deploy Container') {
            steps {
                script {
                    sh "docker run -d -p 8081:9090 ${DOCKER_IMAGE}:${DOCKER_TAG}"

                }
            }
        }
    }
}
