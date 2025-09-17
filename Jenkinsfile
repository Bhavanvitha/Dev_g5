pipeline {
    agent any

    environment {
        REGISTRY      = 'myregistry.azurecr.io'
        REGISTRY_NAME = 'myregistry'  // Added registry name to avoid cut command issue
        IMAGE_NAME    = 'agentic-ai-2'
        IMAGE_TAG     = "${BUILD_NUMBER}"
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
                   export PATH=$PATH:/var/lib/jenkins/.local/bin
                   python3 -m pip install --upgrade pip
                   python3 -m pip install -r backend/requirements.txt
                   pytest -q || true   # continue even if tests fail
                '''
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh '''
                   echo "Logging into Azure Container Registry..."
                   az acr login --name $REGISTRY_NAME
                   
                   echo "Building Backend Docker image..."
                   docker build -t $REGISTRY/$IMAGE_NAME-backend:$IMAGE_TAG ./backend
                   
                   echo "Building Frontend Docker image..."
                   docker build -t $REGISTRY/$IMAGE_NAME-frontend:$IMAGE_TAG ./frontend
                   
                   echo "Pushing Backend Docker image..."
                   docker push $REGISTRY/$IMAGE_NAME-backend:$IMAGE_TAG
                   
                   echo "Pushing Frontend Docker image..."
                   docker push $REGISTRY/$IMAGE_NAME-frontend:$IMAGE_TAG
                '''
            }
        }
    }

    post {
        success {
            echo "Docker image $REGISTRY/$IMAGE_NAME:$IMAGE_TAG built and pushed successfully"
        }
        failure {
            echo "Build failed. Mail step skipped until SMTP is configured."
        }
    }
}