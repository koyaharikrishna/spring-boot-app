pipeline {
    agent any
    
    tools {
        maven 'maven-3.9' 
    }

    environment {
        DOCKER_USER = 'your-dockerhub-username'
        APP_NAME = 'spring-boot-app'
    }

    stages {
        stage('Build JAR') {
            steps {
                // Use 'bat' for Windows instead of 'sh'
                bat 'mvn clean package -DskipTests'
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                script {
                    // Use 'bat' for docker commands too
                    bat "docker build -t %DOCKER_USER%/%APP_NAME%:%BUILD_NUMBER% ."
                    bat "docker tag %DOCKER_USER%/%APP_NAME%:%BUILD_NUMBER% %DOCKER_USER%/%APP_NAME%:latest"
                    
                    // For Docker login on Windows, it's safer to use standard bat commands
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-auth', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        bat "docker login -u %USER% -p %PASS%"
                        bat "docker push %DOCKER_USER%/%APP_NAME%:%BUILD_NUMBER%"
                        bat "docker push %DOCKER_USER%/%APP_NAME%:latest"
                    }
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                // Change 'sh' to 'bat' for kubectl
                bat "kubectl apply -f k8s/deployment.yaml"
                bat "kubectl rollout restart deployment/%APP_NAME%"
            }
        }
    }
}