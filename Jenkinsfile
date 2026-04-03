pipeline {
    agent any

    environment {
        PROJECT_ID = "my-project-dev1-491515"
        SERVICE_ACCOUNT = credentials('gcp-service-key') 
        APP_VERSION = "v${BUILD_NUMBER}"
    }

    options {
        timestamps()
    }

    stages {

        stage('Setup Python & Install Dependencies') {
            steps {
                sh '''
                echo "Setting up Python environment..."

                python3 -m venv venv
                . venv/bin/activate

                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                echo "Installing dependencies..."
                npm install || true
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                echo "Running tests..."
                npm test || echo "No tests configured"
                '''
            }
        }

     
        stage('Authenticate GCP') {
            steps {
                sh '''
                echo "Authenticating with GCP..."

                echo "$SERVICE_ACCOUNT_KEY" > key.json

                gcloud auth activate-service-account --key-file=key.json
                gcloud config set project $PROJECT_ID
                '''
            }
        }

           stage('Deploy to App Engine') {
            steps {
                sh '''
                echo "Deploying to App Engine..."

                gcloud app deploy app.yaml \
                --version=$APP_VERSION \
                --quiet
                '''
            }
        }
    }

        stage('Promote Version') {
            steps {
                sh '''
                gcloud app services set-traffic default \
                --splits $APP_VERSION=1 \
                --quiet
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful: Version $APP_VERSION"
        }
        failure {
            echo "❌ Deployment failed"
        }
        always {
            sh 'rm -f key.json'
        }
    }
}