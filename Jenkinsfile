pipeline {
    agent any

    tools {
        jdk 'jdk17'            // ✅ Must match your Jenkins JDK tool name
        maven 'maven3.8'       // ✅ Must match your Jenkins Maven tool name
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'   // ✅ Must match Sonar Scanner tool name
    }

    stages {
        stage('Git Checkout') {
            steps {
                echo "📥 Cloning GitHub repository..."
                git branch: 'main', url: 'https://github.com/harikrishnauppara11-create/Demo-project.git'
            }
        }

        stage('Compile') {
            steps {
                echo "⚙️ Compiling project..."
                sh 'mvn compile'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "🔍 Running SonarQube analysis..."
                withSonarQubeEnv('sonar-server') {  // ✅ Must match your SonarQube server name
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=poc-10 \
                        -Dsonar.projectName=poc-10 \
                        -Dsonar.java.binaries=target
                    '''
                }
            }
        }

        stage('Build') {
            steps {
                echo "🏗️ Building project..."
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('Docker Build and Push') {
            steps {
                script {
                    echo "🐳 Building and pushing Docker image..."
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh '''
                            docker build -t navya111yadagalla/poc-10:latest .
                            docker push navya111yadagalla/poc-10:latest
                        '''
                    }
                }
            }
        }

        stage('Deploy to Docker Container') {
            steps {
                script {
                    echo "🚀 Deploying container locally..."
                    // Stop and remove old container if it exists
                    sh '''
                        docker stop container || true
                        docker rm container || true
                        docker run -d --name container -p 8081:8081 navya111yadagalla/poc-10:latest
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs for details."
        }
    }
}
