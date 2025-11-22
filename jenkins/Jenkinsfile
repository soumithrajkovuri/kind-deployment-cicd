pipeline {
    agent any

    stages {

        stage('Build') {
            steps {
                sh 'docker build -t myapp:$BUILD_NUMBER app/'
            }
        }

        stage('Push') {
            steps {
                sh 'kind load docker-image myapp:$BUILD_NUMBER --name jenkins-kind'
            }
        }

        stage('Deploy') {
            steps {
                ansiblePlaybook(
                    playbook: 'ansible/playbook-deploy.yaml',
                    extras: "-e image_tag=$BUILD_NUMBER"
                )
            }
        }
    }
}
