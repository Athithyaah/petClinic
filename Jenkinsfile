//Just an empty script for now 
pipeline {
  agent any

  environment {
    PROJECT_ID = "your-gcp-project-id"
    CLUSTER_NAME = "your-gke-cluster"
    GKE_ZONE = "your-cluster-zone"
    REGISTRY = "gcr.io/${PROJECT_ID}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build App') {
      steps {
        sh 'mvn clean package'
      }
    }

    stage('Build Docker Images') {
      steps {
        script {
          docker.build("${REGISTRY}/petclinic-app", "-f Dockerfile .")
          docker.build("${REGISTRY}/jenkins", "-f jenkins/Dockerfile jenkins")
          docker.build("${REGISTRY}/sonarqube", "-f sonarqube/Dockerfile sonarqube")
          docker.build("${REGISTRY}/nexus", "-f nexus/Dockerfile nexus")
        }
      }
    }

    stage('Push Docker Images') {
      steps {
        withCredentials([file(credentialsId: 'gcp-service-account-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
          sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
          sh 'gcloud auth configure-docker'
          sh "docker push ${REGISTRY}/petclinic-app"
          sh "docker push ${REGISTRY}/jenkins"
          sh "docker push ${REGISTRY}/sonarqube"
          sh "docker push ${REGISTRY}/nexus"
        }
      }
    }

    stage('Deploy to GKE') {
      steps {
        withCredentials([file(credentialsId: 'gcp-service-account-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
          sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
          sh "gcloud container clusters get-credentials ${CLUSTER_NAME} --zone ${GKE_ZONE} --project ${PROJECT_ID}"
          sh 'kubectl apply -f k8s/'
        }
      }
    }

    stage('Monitoring Setup') {
      steps {
        sh 'kubectl apply -f monitoring/prometheus.yaml'
        sh 'kubectl apply -f monitoring/grafana.yaml'
      }
    }
  }

  post {
    always {
      echo 'Cleaning up workspace...'
      cleanWs()
    }
  }
}
