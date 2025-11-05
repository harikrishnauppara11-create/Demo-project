pipeline {
    agent any

    environment {
        // 🧩 Replace with your real configuration
        PROJECT_ID = "crucial-quarter-477301-p6"
        IMAGE_NAME = "demo-app"
        SONARQUBE_ENV = "SonarQube servers"   // Name from Jenkins > Manage Jenkins > Configure System
        REGISTRY = "gcr.io/${PROJECT_ID}/${IMAGE_NAME}"
        SONAR_TOKEN = "your_sonarqube_token_here"
        CLUSTER_NAME = "your_gke_cluster_name"
        CLUSTER_ZONE = "your_gke_zone"
    }

    tools {
        maven 'Maven'   // ✅ Must match Jenkins Global Tool Configuration
        jdk 'JDK11'     // ✅ Must match Jenkins JDK name
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
                sh "mvn clean package -DskipTests"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "🔍 Running SonarQube Analysis..."
                withSonarQubeEnv('SonarQube servers') {   // Must match your SonarQube server name
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
