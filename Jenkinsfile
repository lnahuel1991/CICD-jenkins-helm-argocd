pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE_NAME = "lnahuel/node-argocd"
        DOCKER_IMAGE_TAG = "${env.BUILD_NUMBER}"
    }
        
    stages {
        stage('Checkout') {
            steps {
                // Checkout the source code from your Git repository
                git branch: 'main',
                url: 'https://github.com/lnahuel1991/CICD-jenkins-helm-argocd.git'
            }
        }
 
        stage('Remove All Previous Docker Images') {
            steps {
                // Remove all Docker images
                sh 'docker rmi -f $(docker images -a -q) || true'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image with the specified name and tag
                    sh "docker build --rm -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ."
                }
            }
        }

        stage('Push Docker Image to Registry') {
            steps {
                script {
                    // Log in to Docker registry and push the Docker image
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        sh "echo ${DOCKERHUB_PASSWORD} | docker login -u ${DOCKERHUB_USERNAME} --password-stdin"
                        sh "docker push ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                    }
                }
            }
        }
        
        stage('Trigger ManifestUpdate') {
            steps {
                echo "triggering updatemanifestjob"
                build job: 'updatemanifest', parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)]
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline execution successful!'
        }
        failure {
            echo 'Pipeline execution failed!'
        }
    }
}
