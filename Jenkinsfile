pipeline {
    agent any

    environment {
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ncttan-DE/bcakend-docker'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'aws-account-id', variable: 'AWS_ACCOUNT_ID'),
                        string(credentialsId: 'aws-region', variable: 'AWS_REGION'),
                        string(credentialsId: 'bcakend-docker', variable: 'REPO_NAME')
                    ]) {
                        dockerImage = docker.build("${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}:${IMAGE_TAG}")
                    }
                }
            }
        }

        stage('Login to ECR & Push') {
            steps {
                withAWS(credentials: 'aws-ecr-cred', region: "${AWS_REGION}") {
                    echo '''
                        aws ecr get-login-password --region ${AWS_REGION} \
                        | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com

                        docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Image pushed successfully to ECR!"
        }
        failure {
            echo "❌ Build failed."
        }
    }
}
