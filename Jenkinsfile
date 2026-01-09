pipeline {

    agent { label 'jenkins-agent' }

    stages {
        stage('Cleanup & Initialize') {
            steps {
                echo "Checking connection to Kubernetes..."
                // Using --insecure-skip-tls-verify to avoid certificate issues on local Kind
                sh 'kubectl cluster-info --insecure-skip-tls-verify'

                echo "Creating Namespace if it doesn't exist..."
                sh 'kubectl apply -f k8s/dev/namespace.yaml --insecure-skip-tls-verify'

                // Small sleep to let the K8s API register the namespace
                sleep 3
            }
        }

        stage('Deploy Secrets & Infrastructure') {
            steps {
                echo "Applying Postgres..."

                sh 'kubectl apply -f k8s/dev/postgres.yaml --insecure-skip-tls-verify'

                echo "Waiting for Postgres to be ready..."
                // This prevents the Backend from crashing while waiting for the DB
                sh 'kubectl rollout status deployment/retail-postgres -n retail --timeout=60s --insecure-skip-tls-verify || echo "Postgres taking longer than expected..." '
            }
        }

        stage('Deploy Application') {
            steps {
                echo "Deploying Backend and UI..."
                sh 'kubectl apply -f k8s/dev/backend.yaml --insecure-skip-tls-verify'
                sh 'kubectl apply -f k8s/dev/ui.yaml --insecure-skip-tls-verify'
            }
        }

        stage('Verify Deployment') {
            steps {
                echo "Final Status of Pods in 'retail' namespace:"
                sh 'kubectl get pods -n retail --insecure-skip-tls-verify'
            }
        }
    }

    post {
        always {
            echo "Pipeline finished. Check the 'retail' namespace for status."
        }
        failure {
            echo "Pipeline failed. Check the logs above for kubectl errors."
        }
    }
}