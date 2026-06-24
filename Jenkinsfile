pipeline {
  agent any

  environment {
    NAMESPACE = "retail-dev"
  }

  stages {

    stage('Initialize') {
      steps {
        withCredentials([file(credentialsId: 'kubeconfig-retail', variable: 'KUBECONFIG')]) {
          echo "Creating namespaces and service bridges..."
          sh '''
            kubectl get ns ${NAMESPACE} --kubeconfig=$KUBECONFIG >/dev/null 2>&1 \
              || kubectl create ns ${NAMESPACE} --kubeconfig=$KUBECONFIG

            kubectl get ns retail --kubeconfig=$KUBECONFIG >/dev/null 2>&1 \
              || kubectl create ns retail --kubeconfig=$KUBECONFIG

            kubectl -n retail get svc retail-backend --kubeconfig=$KUBECONFIG >/dev/null 2>&1 \
              || kubectl create service externalname retail-backend -n retail \
                   --external-name retail-backend.${NAMESPACE}.svc.cluster.local \
                   --kubeconfig=$KUBECONFIG
          '''
        }
      }
    }

    stage('Deploy Infrastructure') {
      steps {
        withCredentials([file(credentialsId: 'kubeconfig-retail', variable: 'KUBECONFIG')]) {
          echo "Applying secrets, configmap, postgres, redis..."
          sh "kubectl apply -n ${NAMESPACE} -f k8s/dev/secrets.yaml  --kubeconfig=$KUBECONFIG"
          sh "kubectl apply -n ${NAMESPACE} -f k8s/dev/configmap.yaml --kubeconfig=$KUBECONFIG"
          sh "kubectl apply -n ${NAMESPACE} -f k8s/dev/postgres.yaml  --kubeconfig=$KUBECONFIG"
          sh "kubectl apply -n ${NAMESPACE} -f k8s/dev/redis.yaml     --kubeconfig=$KUBECONFIG"

          echo "Waiting for Postgres to be ready..."
          sh "kubectl rollout status deployment/retail-db -n ${NAMESPACE} --timeout=120s --kubeconfig=$KUBECONFIG || true"
          echo "Waiting for Redis to be ready..."
          sh "kubectl rollout status deployment/retail-redis -n ${NAMESPACE} --timeout=60s --kubeconfig=$KUBECONFIG || true"
        }
      }
    }

    stage('Deploy Application') {
      steps {
        withCredentials([file(credentialsId: 'kubeconfig-retail', variable: 'KUBECONFIG')]) {
          echo "Deploying backend, UI, and Ingress..."
          sh "kubectl apply -n ${NAMESPACE} -f k8s/dev/backend.yaml  --kubeconfig=$KUBECONFIG"
          sh "kubectl apply -n ${NAMESPACE} -f k8s/dev/ui.yaml       --kubeconfig=$KUBECONFIG"
          sh "kubectl apply -n ${NAMESPACE} -f k8s/dev/ingress.yaml  --kubeconfig=$KUBECONFIG"

          echo "Rolling restart to pick up latest image..."
          sh "kubectl -n ${NAMESPACE} rollout restart deployment/retail-backend --kubeconfig=$KUBECONFIG || true"
          sh "kubectl -n ${NAMESPACE} rollout restart deployment/retail-ui      --kubeconfig=$KUBECONFIG || true"

          echo "Waiting for backend rollout..."
          sh "kubectl rollout status deployment/retail-backend -n ${NAMESPACE} --timeout=180s --kubeconfig=$KUBECONFIG"
        }
      }
    }

    stage('Verify') {
      steps {
        withCredentials([file(credentialsId: 'kubeconfig-retail', variable: 'KUBECONFIG')]) {
          sh "kubectl get pods    -n ${NAMESPACE} --kubeconfig=$KUBECONFIG"
          sh "kubectl get svc     -n ${NAMESPACE} --kubeconfig=$KUBECONFIG"
          sh "kubectl get ingress -n ${NAMESPACE} --kubeconfig=$KUBECONFIG"
        }
      }
    }
  }

  post {
    failure {
      withCredentials([file(credentialsId: 'kubeconfig-retail', variable: 'KUBECONFIG')]) {
        echo "Pipeline failed — rolling back backend and UI deployments..."
        sh "kubectl rollout undo deployment/retail-backend -n ${NAMESPACE} --kubeconfig=$KUBECONFIG || true"
        sh "kubectl rollout undo deployment/retail-ui      -n ${NAMESPACE} --kubeconfig=$KUBECONFIG || true"
        sh "kubectl get pods -n ${NAMESPACE} --kubeconfig=$KUBECONFIG"
        echo "Rollback complete. Check logs with: kubectl logs -n ${NAMESPACE} deployment/retail-backend"
      }
    }
    success {
      echo "Deployment successful — ${NAMESPACE} is live."
    }
  }
}
