pipeline {
    agent any

    options {
        skipDefaultCheckout true
    }

    environment {
        PATH = "/usr/bin:/usr/local/bin:${env.PATH}"
        REGISTRY = "localhost:5000"
        IMAGE_NAME = "myapp"
        IMAGE_TAG = "v${BUILD_NUMBER}"
        RELEASE_NAME = "myapp"
        CHART_PATH = "helm/myapp"
    }

    stages {

        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ./app
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                sh """
                    docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy via Ansible') {
            steps {
                sh """
                    /usr/bin/python3 -m ansible.cli.playbook ansible/playbook-deploy.yaml -e image_tag=${IMAGE_TAG}
                """
            }
        }

    }

    post {
        success { echo "Deployment successful!" }
        failure { echo "Build failed!" }
    }
}
