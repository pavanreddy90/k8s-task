pipeline {
    agent any
    environment {
        PROJECT_ID = "${env.PROJECT_ID}"
        GCR_REGION = 'us-central1'
        IMAGE_NAME = "gcr.io/${PROJECT_ID}/sample-app"
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/pavanreddy90/k8s-task.git'
            }
        }
        stage('Build') {
            steps {
                script {
                    dockerImage = docker.build("${IMAGE_NAME}:${env.BUILD_ID}")
                }
            }
        }
        stage('Push') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'gcp-sa.key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                        sh 'gcloud auth configure-docker --quiet'
                        docker.withRegistry("https://gcr.io", '') {
                            dockerImage.push("${env.BUILD_ID}")
                            dockerImage.push("latest")
                        }
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'gcp-sa.key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                        sh 'gcloud container clusters get-credentials devops-task-cluster --zone us-central1-a --project ${PROJECT_ID}'
                        sh 'kubectl apply -f k8s/deploy.yaml'
                        sh 'kubectl apply -f k8s/svc.yaml'
                    }
                }
            }
        }
    }
    post {
        success {
            echo 'Deployment was successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
