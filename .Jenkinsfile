pipeline {
    agent any
    environment {
        PROJECT_ID = "${env.PROJECT_ID}" // GCP Project ID stored in an environment variable
        GCR_REGION = 'us-central1' // GCR Region
        IMAGE_NAME = "gcr.io/${PROJECT_ID}/sample-app" // Name of the Docker image
        GOOGLE_APPLICATION_CREDENTIALS = "${env.GCP_SA_KEY}" // Path to the Service Account key file stored in an environment variable
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
                    sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                    sh 'gcloud auth configure-docker --quiet'
                    docker.withRegistry("https://gcr.io", '') {
                        dockerImage.push("${env.BUILD_ID}")
                        dockerImage.push("latest")
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    // Authenticate to GKE and deploy the Docker image to the Kubernetes cluster
                    sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                    sh 'gcloud container clusters get-credentials devops-task-cluster --zone us-central1-a --project $PROJECT_ID'
                    sh 'kubectl apply -f k8s/deploy.yaml'
                    sh 'kubectl apply -f k8s/svc.yaml'
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
