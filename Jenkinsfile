pipeline {
  agent any

  environment {
    NAMESPACE = "retail"
    KUBE_CONTEXT = "docker-desktop"
    // Updated to the standard Jenkins home path
    KUBECONFIG = "/var/jenkins_home/.kube/config"
  }

  stages {
    stage('Checkout infra repo') {
      steps { checkout scm }
    }

    stage('K8s Deployment') {
      steps {
        sh """
          # Infrastructure
          kubectl --kubeconfig=${KUBECONFIG} --context ${KUBE_CONTEXT} apply -f k8s/dev/namespace.yaml
          kubectl --kubeconfig=${KUBECONFIG} --context ${KUBE_CONTEXT} apply -f k8s/dev/postgres.yaml

          # Application
          kubectl --kubeconfig=${KUBECONFIG} --context ${KUBE_CONTEXT} apply -f k8s/dev/backend.yaml
          kubectl --kubeconfig=${KUBECONFIG} --context ${KUBE_CONTEXT} apply -f k8s/dev/ui.yaml

          # Force Refresh & Status check
          kubectl --kubeconfig=${KUBECONFIG} --context ${KUBE_CONTEXT} rollout restart deploy/retail-backend deploy/retail-ui -n ${NAMESPACE}
          kubectl --kubeconfig=${KUBECONFIG} --context ${KUBE_CONTEXT} rollout status deploy/retail-backend -n ${NAMESPACE} --timeout=180s
        """
      }
    }
  }
}