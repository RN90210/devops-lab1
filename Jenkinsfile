pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_URI = 'PASTE_YOUR_ECR_URI_HERE'
    }

    stages {
        stage('Set Image Tag') {
            steps {
                script {
                    env.IMAGE_TAG = sh(
                        script: 'git rev-parse --short=7 HEAD',
                        returnStdout: true
                    ).trim()
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv .venv
                    .venv/bin/pip install --upgrade pip
                    .venv/bin/pip install flake8 pytest
                '''
            }
        }

        stage('Lint') {
            steps {
                sh '.venv/bin/flake8 app.py'
            }
        }

        stage('Unit Tests') {
            steps {
                sh '.venv/bin/pytest test_app.py -v'
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin $ECR_URI
                '''
            }
        }

        stage('Build Image') {
            steps {
                sh '''
                    docker build -t $ECR_URI:$IMAGE_TAG .
                    docker tag $ECR_URI:$IMAGE_TAG $ECR_URI:latest
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                    docker push $ECR_URI:$IMAGE_TAG
                    docker push $ECR_URI:latest
                '''
            }
        }
    }

    post {
        success {
            echo "SUCCESS: Image pushed as ${ECR_URI}:${IMAGE_TAG}"
        }

        failure {
            echo "FAILED: Check the stage where the pipeline stopped."
        }
    }
}
