pipeline {
    agent { label 'jenkins-agent' }

    stages {
        stage('Initialize & Deploy') {
            steps {
                // No container() block needed!
                // We use the 'jnlp' container which is already active.
                sh 'kubectl version --client --insecure-skip-tls-verify'
                sh 'kubectl apply -f k8s/dev/backend.yaml --insecure-skip-tls-verify'
            }
        }
    }
}