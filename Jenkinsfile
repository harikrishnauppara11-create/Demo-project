pipeline {
    agent any

    tools {
        // Must match Jenkins global tool configuration names
        jdk 'JDK11'
        maven 'Maven'
    }

    environment {
        PROJECT_ID = 'crucial-quarter-477301-p6'
        IMAGE_NAME = 'demo-app'
        IMAGE_TAG = 'latest'
        GCR_REGION = 'asia-south1'
        GKE_CLUSTER = 'gke-cluster'          // replace with your actual GKE cluster name
        GKE_ZONE = 'asia-south1-a'           // replace with your GKE zone
        SONARQUBE_ENV = 'SonarQube servers'  // must match your configured name
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo '📥 Checking out source code...'
                git branch: 'main', url: 'https://github.com/harikrishnauppara11-create/Demo-project.git'
            }
        }

        stage('Build with Maven') {
            steps {
                echo '⚙️ Building project with Maven...'
                sh 'mvn -B clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo '🔍 Running SonarQube Analysis...'
                withSonarQubeEnv('SonarQube servers') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image...'
                sh """
                    docker build -t gcr.io/${PROJECT_ID}/${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Push Docker Image to GCR') {
            steps {
                echo '📤 Pushing Docker image to GCR...'
                sh """
                    gcloud auth configure-docker ${GCR_REGION}-docker.pkg.dev --quiet
                    docker push gcr.io/${PROJECT_ID}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy to GKE') {
            steps {
                echo '🚀 Deploying application to GKE...'
                sh """
                    gcloud container clusters get-credentials ${GKE_CLUSTER} --zone ${GKE_ZONE} --project ${PROJECT_ID}
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                """
            }
        }
    }

    post {
        success {
            echo '✅ Deployment successful!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs for errors.'
        }
    }
}
