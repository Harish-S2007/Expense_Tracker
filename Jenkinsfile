
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Tools') {
            steps {
                powershell '''
                    python --version
                    docker --version
                    kubectl version --client
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                powershell '''
                    python -m pip install -r requirements.txt
                    if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }
                '''
            }
        }

        stage('Test Application') {
            steps {
                powershell '''
                    python -c "import app; print('Application loaded successfully')"
                    if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                powershell '''
                    docker build -t expense-tracker:v1 .
                    if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                powershell '''
                    kubectl apply -f deployment.yaml
                    if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }

                    kubectl rollout status deployment/expense-tracker --timeout=180s
                    if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                powershell '''
                    kubectl get deployments
                    kubectl get pods
                    kubectl get services
                '''
            }
        }
    }

    post {
        success {
            echo 'Expense Tracker V1 pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
