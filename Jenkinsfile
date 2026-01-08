pipeline {
    agent {

        label 'jenkins-agent'
    }

    stages {
        stage('Initialize') {
            steps {

                sh 'kubectl version --client'
                echo "Running on a dynamic Kubernetes agent!"
            }
        }

        stage('Deploy Application') {
            steps {

                container('kubectl') {
                    sh 'kubectl apply -f k8s/dev/backend.yaml'
                    sh 'kubectl rollout status deployment/retail-backend -n retail'
                }
            }
        }
    }
}