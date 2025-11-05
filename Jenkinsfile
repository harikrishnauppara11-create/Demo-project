pipeline {
    agent any

    environment {
        // 🔧 Replace with your real details
        PROJECT_ID = "crucial-quarter-477301-p6"
        IMAGE_NAME = "demo-app"
        SONARQUBE = "SonarQube servers"  // Jenkins → Manage Jenkins → Configure System → SonarQube name
        REGISTRY = "gcr.io/${PROJECT_ID}/${IMAGE_NAME}"
        SONAR_TOKEN = "your_sonarqube_token_here"
        CLUSTER_NAME = "your_gke_cluster_name"
        CLUSTER_ZONE = "your_gke_zone"
    }

    tools {
        maven 'Maven'   // ✅ Name must exactly match the one in Jenkins Tool config
        jdk 'JDK11'
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "📥 Cloning GitHub repository..."
                git branch: 'main', url: 'https://github.com/harikrishnauppara11-create/Demo-project.git'
            }
        }

        stage('Build with Maven') {
            steps {
                echo "🏗️ Building project using Maven..."
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "🔍 Running SonarQube Analysis..."
                withSonarQubeEnv('SonarQube servers') {   // ✅ must match Jenkins SonarQube name
                    sh """
                        mvn sonar:sonar \
                          -Dsonar.projectKey=demo-project \
                          -Dsonar.host.url=http://34.14.169.31:9000 \
                          -Dsonar.login=$SONAR_TOKEN
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image..."
                sh 'docker build -t $REGISTRY:$BUILD_NUMBER .'
            }
        }

        stage('Push Docker Image to GCR') {
            steps {
                echo "☁️ Pushing Docker image to Google Container Registry..."
                sh """
                    gcloud auth configure-docker --quiet
                    docker push $REGISTRY:$BUILD_NUMBER
                """
            }
        }

        stage('Deploy to GKE') {
            steps {
                echo "🚀 Deploying application to GKE..."
                sh """
                    gcloud container clusters get-credentials $CLUSTER_NAME --zone $CLUSTER_ZONE --project $PROJECT_ID
                    kubectl set image deployment/demo-app demo-app=$REGISTRY:$BUILD_NUMBER
                """
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs for errors."
        }
    }
}
