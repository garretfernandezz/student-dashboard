pipeline {
    agent any

    environment {
        DOCKERHUB_CRED = credentials('dockerhub-creds')
        IMAGE_NAME = "yourusername/student-dashboard"
        NAMESPACE = "${env.BRANCH_NAME == 'main' ? 'production' : 'test'}"
        TAG = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Replace placeholder in HTML
                    sh "sed -i 's/{{BUILD_NUMBER}}/${BUILD_NUMBER}/g' app/index.html"

                    dockerImage = docker.build("${IMAGE_NAME}:${TAG}")
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    docker.withRegistry('', 'dockerhub-creds') {
                        dockerImage.push()
                        dockerImage.push("${env.BRANCH_NAME}-latest")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Replace placeholders in YAML
                    sh """
                    sed "s|NAMESPACE|${NAMESPACE}|g; s|TAG|${TAG}|g" deployment.yaml > deployment-rendered.yaml
                    """

                    // Apply using kubectl
                    sh "kubectl apply -f deployment-rendered.yaml"
                }
            }
        }
    }

    post {
        success {
            echo "Deployed to ${NAMESPACE} with tag ${TAG}"
        }
        cleanup {
            sh "docker rmi ${IMAGE_NAME}:${TAG} || true"
        }
    }
}