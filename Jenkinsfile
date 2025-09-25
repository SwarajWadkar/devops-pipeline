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
                sh '''#!/bin/bash
                echo $DOCKER_HUB_PASS | docker login -u $DOCKER_HUB_USER --password-stdin
                docker push $DOCKER_HUB_USER/devops-app:$BUILD_NUMBER
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''#!/bin/bash
                kubectl apply -f deployment.yaml
                kubectl apply -f service.yaml
                '''
            }
        }
    }

    post {
        always {
            sh '''#!/bin/bash
            docker logout
            '''
        }
    }
}
