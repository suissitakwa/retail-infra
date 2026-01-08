pipeline {
    agent {
        label 'jenkins-agent'
    }

    stages {
        stage('Initialize') {
            steps {
                // Move inside the kubectl container so the command is found
                container('kubectl') {
                    sh 'kubectl version --client'
                }
                echo "Running on a dynamic Kubernetes agent!"
            }
        }

        stage('Deploy Application') {
            steps {
                container('kubectl') {
                    // Added the insecure flag to handle the certificate issue we saw earlier
                    sh 'kubectl apply -f k8s/dev/backend.yaml --insecure-skip-tls-verify'
                    sh 'kubectl rollout status deployment/retail-backend -n retail --insecure-skip-tls-verify'
                }
            }
        }
    }
}