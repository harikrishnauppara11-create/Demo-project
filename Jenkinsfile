pipeline {
    agent any

    environment {
        PROJECT_ID = 'crucial-quarter-477301-p6'
        IMAGE_NAME = 'demo-project'
        IMAGE_TAG = "v${BUILD_NUMBER}"
        SONAR_PROJECT_KEY = 'demo-project'
        SONAR_HOST_URL = 'http://34.14.169.31:9000'
        SONARQUBE_ENV = 'SonarQube servers'
        SONAR_TOKEN = credentials('sonar-token')       // Use the credential you added in Jenkins
        GCP_SA = credentials('gcp-service-account')    // GCP service account key (JSON)
    }

    tools {
        maven 'Maven'
        jdk 'JDK11'
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo '📦 Checking out code from GitHub...'
                git branch: 'main', url: 'https://github.com/harikrishnauppara11-create/Demo-project.git'
            }
        }

        stage('Build with Maven') {
            steps {
                echo '🔨 Building application with Maven...'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo '🔍 Running SonarQube Analysis...'
                withSonarQubeEnv('SonarQube servers') {
                    sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=${SONAR_TOKEN}
                    """
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
                echo '🚀 Pushing Docker image to Google Container Registry...'
                withCredentials([file(credentialsId: 'gcp-service-account', variable: 'GCP_KEY')]) {
                    sh """
                        gcloud auth activate-service-account --key-file=$GCP_KEY
                        gcloud auth configure-docker gcr.io -q
                        docker push gcr.io/${PROJECT_ID}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                echo '☸️ Deploying to Google Kubernetes Engine...'
                withCredentials([file(credentialsId: 'gcp-service-account', variable: 'GCP_KEY')]) {
                    sh """
                        gcloud auth activate-service-account --key-file=$GCP_KEY
                        gcloud container clusters get-credentials my-gke-cluster --zone us-central1-a --project ${PROJECT_ID}
                        kubectl set image deployment/${IMAGE_NAME}-deployment ${IMAGE_NAME}=gcr.io/${PROJECT_ID}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline executed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs for errors.'
        }
    }
}
