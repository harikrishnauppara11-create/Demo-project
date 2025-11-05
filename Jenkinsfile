pipeline {
    agent any

    tools {
        jdk 'JDK17'            // Replace with the exact name from Jenkins Global Tool Configuration
        maven 'Maven 3.8.8'    // Replace with the exact Maven name from Jenkins Global Tool Configuration
    }

    environment {
        SCANNER_HOME = tool 'SonarScanner'   // Make sure Sonar Scanner is installed and named exactly in Jenkins
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/navya111yadagalla/mock-project.git'
            }
        }

        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {   // Make sure this matches your SonarQube server name in Jenkins
                    sh """$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=poc-10 \
                        -Dsonar.projectKey=poc-10 \
                        -Dsonar.java.binaries=. """
                }
            }
        }

        stage('Build') {
            steps {
                sh "mvn clean install"
            }
        }

        stage('Docker Build and Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {  // Make sure docker-cred exists in Jenkins Credentials
                        sh "docker build -t poc-10 ."
                        sh "docker tag poc-10 navya111yadagalla/poc-10:latest"
                        sh "docker push navya111yadagalla/poc-10:latest"
                    }
                }
            }
        }

        stage('Deploy to Docker Container') {
            steps {
                script {
                    // Remove any existing container
                    sh "docker rm -f container || true"
                    // Run the new container
                    sh "docker run -d --name container -p 8081:8081 navya111yadagalla/poc-10:latest"
                }
            }
        }

    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
