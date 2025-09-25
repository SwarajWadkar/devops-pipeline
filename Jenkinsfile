pipeline {
  agent any
  environment {
    IMAGE = "swarajwadkar/myapp"        // ✅ Docker Hub username
    DOCKERHUB_CREDS = 'dockerhub-creds' // ✅ Credential ID in Jenkins
  }
  options {
        shell('/bin/bash')   // 👈 force bash
       }
  stages {
    stage('Build Docker Image') {
      steps {
        sh 'docker build -t ${IMAGE}:${GIT_COMMIT::8} .'
      }
    }
    stage('Scan Docker Image') {
      steps {
        sh '''
        trivy image --exit-code 1 --severity CRITICAL,HIGH ${IMAGE}:${GIT_COMMIT::8} || { 
          echo "VULNERABILITIES FOUND"; 
          exit 1; 
        }
        '''
      }
    }
    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh 'echo $DOCKER_PASS | docker login --username $DOCKER_USER --password-stdin'
          sh 'docker push ${IMAGE}:${GIT_COMMIT::8}'
        }
      }
    }
    stage('Deploy to Kubernetes') {
      steps {
        sh '''
        kubectl set image deployment/myapp myapp=${IMAGE}:${GIT_COMMIT::8} --namespace default || kubectl apply -f k8s/deployment.yaml
        '''
      }
    }
  }
  post {
    always {
      sh 'docker logout || true'
    }
  }
}
