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

        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/soumithrajkovuri/kind-deployment-cicd',
                        credentialsId: 'github-secret'
                    ]]
                ])
                sh 'ls -R'
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

        stage('Deploy via Ansible (container)') {
            steps {
                sh """
                    docker run --rm \
                        -v ${WORKSPACE}:/work \
                        -v /root/.kube:/root/.kube \
                        -w /work \
                        python:3.11-slim /bin/sh -c "pip install --no-cache-dir ansible && python -m ansible.cli.playbook ansible/playbook-deploy.yaml -e image_tag=${IMAGE_TAG}"
                """
            }
        }
    }

    post {
        success { echo "Deployment successful!" }
        failure { echo "Build failed!" }
    }
}
