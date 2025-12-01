pipeline {
    agent any

    options {
        skipDefaultCheckout true     
    }

    environment {
        REGISTRY = "localhost:5000"
        IMAGE_NAME = "myapp"
        IMAGE_TAG = "v${BUILD_NUMBER}"
        RELEASE_NAME = "myapp"
        CHART_PATH = "helm/myapp"
    }

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'github-secret', url: 'https://github.com/soumithrajkovuri/kind-deployment-cicd'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Deploy via Ansible') {
            steps {
                sh """
                    ansible-playbook ansible/playbook-deploy.yaml -e image_tag=${IMAGE_TAG}
                """
            }
        }
    }

    post {
        success { echo "Deployment successful!" }
        failure { echo "Build failed!" }
    }
}

