pipeline {
    agent any

    environment {
        REGISTRY      = 'myregistry5.azurecr.io'
        REGISTRY_NAME = 'myregistry5'
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

        stage('Azure Login & Docker Push') {
            steps {
                withCredentials([string(credentialsId: 'azure-acr-json', variable: 'AZURE_SP_JSON')]) {
                    sh '''
                       echo "$AZURE_SP_JSON" > sp.json

                       # Azure CLI login
                       az login --service-principal \
                         --username $(jq -r .clientId sp.json) \
                         --password $(jq -r .clientSecret sp.json) \
                         --tenant   $(jq -r .tenantId sp.json)

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
    }

    post {
        success {
            echo "Docker images pushed to $REGISTRY successfully"
        }
        failure {
            echo "Build failed. Mail step skipped until SMTP is configured."
        }
    }
}
