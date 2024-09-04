pipeline {
    agent any
    environment {
        PROJECT_ID = "${env.PROJECT_ID}" // GCP Project ID stored in an environment variable
        GCR_REGION = 'us-central1' // GCR Region
        IMAGE_NAME = "gcr.io/${PROJECT_ID}/sample-app" // Name of the Docker image
        GOOGLE_APPLICATION_CREDENTIALS = '/tmp/gcp-key.json' // Path to the Service Account key file in Jenkins workspace
    }
    stages {
        stage('Checkout') {
            steps {
                // Pull the latest code from the GitHub repository
                git 'https://github.com/pavanreddy90/k8s-task.git'
            }
        }
        stage('Build') {
            steps {
                script {
                    // Build the Docker image using the Dockerfile in the root directory
                    dockerImage = docker.build("${IMAGE_NAME}:${env.BUILD_ID}")
                }
            }
        }
        stage('Push') {
            steps {
                script {
                    // Authenticate to GCP and push the Docker image to Google Container Registry (GCR)
                    withCredentials([file(credentialsId: 'devops-mf-434605-c949a515702f.json', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
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
                    // Authenticate to GKE and deploy the Docker image to the Kubernetes cluster
                    withCredentials([file(credentialsId: 'devops-mf-434605-c949a515702f.json', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                        sh 'gcloud container clusters get-credentials devops-task-cluster --zone us-central1-a --project $PROJECT_ID'
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
