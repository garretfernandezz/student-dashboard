// === FORCE GIT + WINDOWS FIX ===
def gitCmd = 'C:\\Program Files\\Git\\cmd\\git.exe'
def git(cmd) {
    bat "${gitCmd} ${cmd}"
}

pipeline {
    agent any

    environment {
        DOCKERHUB_CRED = credentials('dockerhub-creds')
        IMAGE_NAME = "garretfernandezz/student-dashboard"
        NAMESPACE = "${env.BRANCH_NAME == 'main' ? 'production' : 'test'}"
        TAG = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
    }

    stages {
        stage('Manual Checkout') {
            steps {
                script {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: env.BRANCH_NAME]],
                        userRemoteConfigs: [[
                            url: 'https://github.com/garretfernandezz/student-dashboard.git',
                            credentialsId: 'github-creds'
                        ]],
                        extensions: [[$class: 'CleanBeforeCheckout']]
                    ])
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    powershell """
                    (Get-Content app\\index.html) -replace '\\{\\{BUILD_NUMBER\\}\\}', '${BUILD_NUMBER}' | Set-Content app\\index.html
                    """
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
                    powershell """
                    ((Get-Content deployment.yaml) -replace 'NAMESPACE', '${NAMESPACE}') -replace 'TAG', '${TAG}' | Set-Content deployment-rendered.yaml
                    kubectl apply -f deployment-rendered.yaml
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployed to ${NAMESPACE} with tag ${TAG}"
        }
    }
}
