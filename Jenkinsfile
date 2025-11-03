pipeline {
    agent any

    stages {
        stage('Demo - It Works!') {
            steps {
                echo "Pipeline is RUNNING!"
                echo "Branch: ${env.BRANCH_NAME}"
                echo "Build #: ${env.BUILD_NUMBER}"
                echo "SUCCESS: CI/CD Pipeline is ALIVE!"
            }
        }
    }

    post {
        success {
            echo "DEMO COMPLETE - JENKINS IS WORKING!"
        }
    }
}
