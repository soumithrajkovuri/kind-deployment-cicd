pipeline {
    agent any

    environment {
        DOCKERHUB_CRED = 'Docker-cred'      // username/password for Docker Hub
        GITHUB_CRED    = 'git-password'        // PAT token credential for Git push
        IMAGE          = "soumith30/myapp"
        TAG            = "v${BUILD_NUMBER}"
        VALUES_FILE    = "helm/myapp/Values.yaml"
    }

    stages {

        stage('Checkout Code') {
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
                withCredentials([usernamePassword(
                    credentialsId: DOCKERHUB_CRED,
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh """
                        echo "${PASS}" | docker login -u "${USER}" --password-stdin
                        docker push ${IMAGE}:${TAG}
                    """
                }
            }
        }

        stage('Update Image Tag in Git (GitOps)') {
            steps {
                // modify values.yaml
                sh """
                    sed -i 's/tag:.*/tag: ${TAG}/' ${VALUES_FILE}

                    git config user.name "jenkins"
                    git config user.email "jenkins@ci"

                    git add ${VALUES_FILE}
                    git commit -m "Update image tag to ${TAG}"
                """

                // push commit to GitHub using PAT
                withCredentials([usernamePassword(
                    credentialsId: GITHUB_CRED,
                    usernameVariable: 'USER',
                    passwordVariable: 'TOKEN'
                )]) {
                    sh """
                        git remote set-url origin https://${USER}:${TOKEN}@github.com/soumithrajkovuri/kind-deployment-cicd.git
                        git push origin HEAD:main
                    """
                }
            }
        }
    }

    post {
        success {
            echo "CI pipeline completed — ArgoCD will deploy automatically!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
