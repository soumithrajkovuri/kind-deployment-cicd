pipeline {
    agent any

    options {
        skipDefaultCheckout true
    }

    environment {
        PATH          = "/usr/bin:/usr/local/bin:${env.PATH}"
        REGISTRY      = "localhost:5000"
        IMAGE_NAME    = "myapp"
        IMAGE_TAG     = "v${BUILD_NUMBER}"
        DOCKERHUB_CRED = 'Docker-cred'
        GITHUB_CRED = 'git-password'
        IMAGE = "soumith30/myapp"
        TAG = "v${BUILD_NUMBER}"
        VALUES_FILE = "helm/myapp/values.yaml"
    }

    stages {

        
        stage('Checkout') {
            steps {
                checkout(scm)
                sh "ls -R"
                checkout scm
                sh "ls -R"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ./app
                """
                sh """
                    docker build -t ${IMAGE}:${TAG} ./app
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                sh """
                    docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
                withCredentials([usernamePassword(credentialsId: DOCKERHUB_CRED,
                        usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh """
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                        docker push ${IMAGE}:${TAG}
                    """
                }
            }
        }

        stage('Deploy via Ansible (container)') {
        stage('Update Image Tag in Git (GitOps)') {
            steps {
                sh """
                    docker run --rm \
                        -v ${WORKSPACE}:/work \
                        -v /root/.kube:/root/.kube \
                        -w /work \
                        python:3.11-slim /bin/bash -c '
                            pip install ansible &&
                            ansible-playbook ansible/playbook-deploy.yaml -e image_tag=${IMAGE_TAG}
                        '
                """
                    sed -i 's/tag:.*/tag: ${TAG}/' ${VALUES_FILE}

                    git config user.name "jenkins"
                    git config user.email "jenkins@ci"

                    git add ${VALUES_FILE}
                    git commit -m "Update image tag to ${TAG}"
                """

                sshagent(credentials: ['git-password']) {
                    sh """
                        git push origin main
                    """
                }
            }
        }
    }

    post {
        success { echo "Deployment successful!" }
        failure { echo "Deployment failed!" }
        success { echo "CI done — ArgoCD will deploy automatically." }
    }
}
