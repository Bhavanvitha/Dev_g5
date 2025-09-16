pipeline {
    agent any

    environment {
        REGISTRY   = 'myregistry.azurecr.io'
        IMAGE_NAME = 'agentic-ai-2'
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Bhavanvitha/Dev_g5.git'
            }
        }

        stage('Install & Test') {
            steps {
                sh '''
                   python3 -m venv venv
                   . venv/bin/activate && pip install -r requirements.txt && pytest -q
                '''
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh '''
                   REGISTRY_NAME=$(echo $REGISTRY | cut -d'.' -f1)
                   az acr login --name $REGISTRY_NAME
                   docker build -t $REGISTRY/$IMAGE_NAME:$IMAGE_TAG .
                   docker push $REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
    }

    post {
        success {
            echo "Docker image $REGISTRY/$IMAGE_NAME:$IMAGE_TAG built and pushed successfully"
        }
        failure {
            mail to: 'bhavanvithakandukuri@gmail.com',
                 subject: "Jenkins Build Failed: ${env.JOB_NAME}",
                 body: "Check console output at ${env.BUILD_URL}"
        }
    }
}
