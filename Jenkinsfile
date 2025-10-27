pipeline {
    agent any

    environment {
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        REPO_NAME = "bcakend-docker"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ncttan-DE/bcakend-docker'
            }
        }

        // stage('Build Docker Image') {
        //     steps {
        //         script {
        //             // Get AWS info from Jenkins global credentials
        //             withCredentials([
        //                 string(credentialsId: 'aws-account-id', variable: 'AWS_ACCOUNT_ID'),
        //                 string(credentialsId: 'aws-region', variable: 'AWS_REGION')
        //             ]) {
        //                 def dockerImage = docker.build("${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}:${IMAGE_TAG}")
        //             }
        //         }
        //     }
        // }

        stage('Login to ECR & Push') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws-account-id', variable: 'AWS_ACCOUNT_ID'),
                    string(credentialsId: 'aws-region', variable: 'AWS_REGION'),
                    [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-ecr-cred']
                ]) {
                    sh '''
                        echo "🔧 Building Docker image..."
                        docker build -t $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/bcakend-docker:$BUILD_NUMBER .

                        echo "🔐 Logging in to ECR..."
                        aws ecr get-login-password --region $AWS_REGION | \
                        docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

                        echo "🚀 Pushing image to ECR..."
                        docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/bcakend-docker:$BUILD_NUMBER
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
