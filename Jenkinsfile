pipeline {
    agent any
    
    // This matches the names you gave in "Manage Jenkins > Tools"
    tools {
        maven 'maven-3.9' 
        dockerTool 'docker-latest'
    }

    environment {
        DOCKER_USER = 'koyaharikrishna'
        APP_NAME = 'spring-boot-app'
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${env.BUILD_NUMBER}" // Use Jenkins build number as version
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/koyaharikrishna/spring-boot-app'
            }
        }

        stage('Build JAR') {
            steps {
                // Clean and package the Spring Boot app
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    // Build the image
                    dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                    
                    // Login and Push to Docker Hub
                    docker.withRegistry('', 'dockerhub-auth') {
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                // Use the Kubeconfig stored in Jenkins to update the cluster
                withKubeConfig([credentialsId: 'k8s-config']) {
                    sh "kubectl apply -f k8s/deployment.yaml"
                    sh "kubectl apply -f k8s/service.yaml"
                    // Force K8s to pull the new 'latest' image
                    sh "kubectl rollout restart deployment/${APP_NAME}"
                }
            }
        }
    }
}