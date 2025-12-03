pipeline {
    agent any

    environment {
        DOCKERHUB_CRED = 'Docker-cred'
        GITHUB_CRED = 'git-password'
        IMAGE = "soumith30/myapp"
        TAG = "v${BUILD_NUMBER}"
        VALUES_FILE = "helm/myapp/values.yaml"
    }

    stages {
        
        stage('Checkout') {
            steps {
                checkout scm
                sh "ls -R"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${IMAGE}:${TAG} ./app
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKERHUB_CRED,
                        usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh """
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                        docker push ${IMAGE}:${TAG}
                    """
                }
            }
        }

        stage('Update Image Tag in Git (GitOps)') {
            steps {
                sh """
                    sed -i 's/tag:.*/tag: ${TAG}/' ${VALUES_FILE}

                    git config user.name "jenkins"
                    git config user.email "jenkins@ci"

                    git add ${VALUES_FILE}
                    git commit -m "Update image tag to ${TAG}"
                """

                sshagent(credentials: ['github-secret']) {
                    sh """
                        git push origin main
                    """
                }
            }
        }
    }

    post {
        success { echo "CI done — ArgoCD will deploy automatically." }
    }
}
