pipeline {
    agent any

    tools {
        // Ensure this matches the name in Manage Jenkins -> Tools
        maven 'maven-3.9'
    }

    environment {
        DOCKER_USER = 'your-dockerhub-username' // Replace with your username
        APP_NAME = 'spring-boot-app'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/koyaharikrishna/spring-boot-app.git'
            }
        }

        stage('Build JAR') {
            steps {
                bat 'mvn clean package -DskipTests -e'
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    // Building the image
                    bat "docker build -t %DOCKER_USER%/%APP_NAME%:latest ."
                    
                    // Logging in and pushing
                    // Note: Create a 'dockerhub-auth' credential in Jenkins (Username/Password)
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-auth', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        bat "docker login -u %USER% -p %PASS%"
                        bat "docker push %DOCKER_USER%/%APP_NAME%:latest"
                    }
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                // This assumes kubectl is configured on the Jenkins Windows server
                bat "kubectl apply -f k8s/deployment.yaml"
            }
        }
    }
}