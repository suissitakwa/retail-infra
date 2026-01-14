pipeline {

  agent { label 'jenkins-agent' }

  environment {
    NAMESPACE = "retail-dev"
  }

  stages {

    stage('Initialize') {
      steps {
        echo "Checking connection to Kubernetes..."
        sh 'kubectl cluster-info'

        echo "Creating Namespace if it doesn't exist..."
        sh '''
          kubectl get ns ${NAMESPACE} >/dev/null 2>&1 || kubectl create ns ${NAMESPACE}
        '''

        sleep 3
      }
    }

    stage('Deploy Infrastructure (Postgres)') {
      steps {
        echo "Applying Postgres..."
        sh 'kubectl apply -n ${NAMESPACE} -f k8s/dev/postgres.yaml'

        echo "Waiting for Postgres to be ready..."
        sh '''
          kubectl -n ${NAMESPACE} rollout status deployment/retail-db --timeout=120s \
          || echo "Postgres taking longer than expected..."
        '''
      }
    }

    stage('Deploy Application') {
      steps {
        echo "Deploying Backend and UI..."
        sh 'kubectl apply -n ${NAMESPACE} -f k8s/dev/backend.yaml'
        sh 'kubectl apply -n ${NAMESPACE} -f k8s/dev/ui.yaml'

        // Force refresh (useful when tag is dev-latest)
        sh '''
          kubectl -n ${NAMESPACE} rollout restart deployment/retail-backend || true
          kubectl -n ${NAMESPACE} rollout restart deployment/retail-ui || true
        '''
      }
    }

    stage('Verify Deployment') {
      steps {
        echo "Final Status of Pods in '${NAMESPACE}' namespace:"
        sh 'kubectl get pods -n ${NAMESPACE} -o wide'
        sh 'kubectl get svc -n ${NAMESPACE} -o wide'
      }
    }
  }

  post {
    always {
      echo "Pipeline finished. Check the '${NAMESPACE}' namespace for status."
    }
    failure {
      echo "Pipeline failed. Check the logs above for kubectl errors."
      sh 'kubectl get events -n ${NAMESPACE} --sort-by=.metadata.creationTimestamp | tail -n 40 || true'
    }
  }
}
