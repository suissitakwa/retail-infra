pipeline {
  agent any

  environment {
    NAMESPACE = "retail-dev"
    // Disables Kafka so the backend doesn't hang/403
    KAFKA_ENABLED = "false"
    SPRING_KAFKA_BOOTSTRAP_SERVERS = ""
  }

  stages {
    stage('Initialize') {
      steps {
        echo "Preparing Namespaces and Service Bridges..."
        sh '''
          # 1. Create main dev namespace
          kubectl get ns ${NAMESPACE} >/dev/null 2>&1 || kubectl create ns ${NAMESPACE}

          # 2. Create 'retail' namespace (needed for UI hardcoded config)
          kubectl get ns retail >/dev/null 2>&1 || kubectl create ns retail

          # 3. Create the ExternalName bridge so UI can find Backend
          kubectl -n retail get svc retail-backend >/dev/null 2>&1 || \
          kubectl create service externalname retail-backend -n retail \
            --external-name retail-backend.${NAMESPACE}.svc.cluster.local
        '''
      }
    }

    stage('Deploy Infrastructure') {
      steps {
        echo "Applying Postgres..."
        sh "kubectl apply -n ${NAMESPACE} -f k8s/dev/postgres.yaml"
      }
    }

    stage('Deploy Application') {
      steps {
        echo "Deploying Backend and UI..."
        # We use 'set env' after apply to ensure our Kafka fixes stick
        sh "kubectl apply -n ${NAMESPACE} -f k8s/dev/backend.yaml"
        sh "kubectl apply -n ${NAMESPACE} -f k8s/dev/ui.yaml"

        sh '''
          kubectl -n ${NAMESPACE} set env deployment/retail-backend \
            KAFKA_ENABLED=${KAFKA_ENABLED} \
            SPRING_KAFKA_BOOTSTRAP_SERVERS=${SPRING_KAFKA_BOOTSTRAP_SERVERS}
        '''

        echo "Forcing image pull for dev-latest..."
        sh "kubectl -n ${NAMESPACE} rollout restart deployment/retail-ui || true"
        sh "kubectl -n ${NAMESPACE} rollout restart deployment/retail-backend || true"
      }
    }

    stage('Verify') {
      steps {
        sh "kubectl get pods -n ${NAMESPACE}"
        echo "App should be live at: http://34.28.225.50"
      }
    }
  }
}