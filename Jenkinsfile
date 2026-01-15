pipeline {
  agent any

  environment {
    NAMESPACE = "retail-dev"
    KAFKA_ENABLED = "false"
    SPRING_KAFKA_BOOTSTRAP_SERVERS = ""
  }

  stages {
    stage('Initialize') {
      steps {

        withCredentials([file(credentialsId: 'kubeconfig-retail', variable: 'KUBECONFIG')]) {
          echo "Preparing Namespaces and Service Bridges..."
          sh '''
            kubectl get ns ${NAMESPACE} --kubeconfig=$KUBECONFIG >/dev/null 2>&1 || kubectl create ns ${NAMESPACE} --kubeconfig=$KUBECONFIG

            kubectl get ns retail --kubeconfig=$KUBECONFIG >/dev/null 2>&1 || kubectl create ns retail --kubeconfig=$KUBECONFIG

            kubectl -n retail get svc retail-backend --kubeconfig=$KUBECONFIG >/dev/null 2>&1 || \
            kubectl create service externalname retail-backend -n retail \
              --external-name retail-backend.${NAMESPACE}.svc.cluster.local --kubeconfig=$KUBECONFIG
          '''
        }
      }
    }

    stage('Deploy Infrastructure') {
      steps {
        withCredentials([file(credentialsId: 'kubeconfig-retail', variable: 'KUBECONFIG')]) {
          echo "Applying Postgres..."
          sh "kubectl apply -n ${NAMESPACE} -f k8s/dev/postgres.yaml --kubeconfig=$KUBECONFIG"
        }
      }
    }

   stage('Deploy Application') {
         steps {
           withCredentials([file(credentialsId: 'kubeconfig-retail', variable: 'KUBECONFIG')]) {
             echo "Deploying Backend and UI..."
             sh "kubectl apply -n ${NAMESPACE} -f k8s/dev/backend.yaml --kubeconfig=$KUBECONFIG"
             sh "kubectl apply -n ${NAMESPACE} -f k8s/dev/ui.yaml --kubeconfig=$KUBECONFIG"

             echo "Applying Kafka Environment Fixes..."

             sh """
               kubectl -n ${env.NAMESPACE} set env deployment/retail-backend \
                 KAFKA_ENABLED=${env.KAFKA_ENABLED} \
                 SPRING_KAFKA_BOOTSTRAP_SERVERS=${env.SPRING_KAFKA_BOOTSTRAP_SERVERS} \
                 --kubeconfig=$KUBECONFIG
             """

             sh "kubectl -n ${env.NAMESPACE} rollout restart deployment/retail-ui --kubeconfig=$KUBECONFIG || true"
             sh "kubectl -n ${env.NAMESPACE} rollout restart deployment/retail-backend --kubeconfig=$KUBECONFIG || true"
           }
         }
       }

    stage('Verify') {
      steps {
        withCredentials([file(credentialsId: 'kubeconfig-retail', variable: 'KUBECONFIG')]) {
          sh "kubectl get pods -n ${NAMESPACE} --kubeconfig=$KUBECONFIG"
        }
      }
    }
  }
}