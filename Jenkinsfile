pipeline {
  agent any

  environment {
    NAMESPACE = "retail"
    KUBE_CONTEXT = "docker-desktop"
  }

  stages {
    stage('Checkout infra repo') {
      steps { checkout scm }
    }

    stage('Use Kube Context') {
      steps {
        sh """
          kubectl config use-context ${KUBE_CONTEXT}
          kubectl get nodes
        """
      }
    }

    stage('Apply namespace + db ') {
      steps {
        sh """
          kubectl apply -f k8s/dev/namespace.yaml
          kubectl apply -f k8s/dev/postgres.yaml

        """
      }
    }

    stage('Deploy backend + ui') {
      steps {
        sh """
          kubectl apply -f k8s/dev/backend.yaml
          kubectl apply -f k8s/dev/ui.yaml
        """
      }
    }

    stage('Rollout') {
      steps {
        sh """
          kubectl rollout status deploy/retail-backend -n ${NAMESPACE} --timeout=180s
          kubectl rollout status deploy/retail-ui -n ${NAMESPACE} --timeout=180s
        """
      }
    }

    stage('Force pull latest images') {
      steps {
        sh """
          kubectl rollout restart deploy/retail-backend -n ${NAMESPACE}
          kubectl rollout restart deploy/retail-ui -n ${NAMESPACE}
          kubectl rollout status deploy/retail-backend -n ${NAMESPACE} --timeout=180s
          kubectl rollout status deploy/retail-ui -n ${NAMESPACE} --timeout=180s
        """
      }
    }
  }
}
