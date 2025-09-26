pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = "swarajwadkar"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                sh '''#!/bin/bash
                docker build -t $DOCKER_HUB_USER/devops-app:$BUILD_NUMBER .
                '''
            }
        }

        stage('Scan Docker Image') {
            steps {
                sh '''#!/bin/bash
                trivy image $DOCKER_HUB_USER/devops-app:$BUILD_NUMBER || true
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', 
                                                 usernameVariable: 'DOCKER_USER', 
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh '''#!/bin/bash
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $DOCKER_HUB_USER/devops-app:$BUILD_NUMBER
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''#!/bin/bash
                kubectl apply -f k8/deployment.yaml
                kubectl apply -f k8/service.yaml

                echo "⏳ Waiting for pods to become ready..."
                kubectl rollout status deployment/devops-app --timeout=120s
                '''
            }
        }
    }

    post {
        always {
            sh '''#!/bin/bash
            docker logout || true
            '''
        }
    }
}
