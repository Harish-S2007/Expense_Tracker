
pipeline {
    agent any

    stages {

        stage('1. Checkout') {
            steps {
                checkout scm
            }
        }

        stage('2. Check Tools') {
            steps {
                powershell '''
                    $python = 'C:\\Users\\hp\\expense_tracker\\.venv\\Scripts\\python.exe'
                    $docker = 'C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
                    $kubectl = 'C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe'

                    Write-Host "Checking Python..."

                    if (-not (Test-Path $python)) {
                        throw "Python executable not found: $python"
                    }

                    & $python --version

                    if ($LASTEXITCODE -ne 0) {
                        throw "Python check failed"
                    }

                    Write-Host "Checking Docker..."

                    if (-not (Test-Path $docker)) {
                        throw "Docker executable not found: $docker"
                    }

                    & $docker --version

                    if ($LASTEXITCODE -ne 0) {
                        throw "Docker check failed"
                    }

                    Write-Host "Checking kubectl..."

                    if (-not (Test-Path $kubectl)) {
                        throw "kubectl executable not found: $kubectl"
                    }

                    & $kubectl version --client

                    if ($LASTEXITCODE -ne 0) {
                        throw "kubectl check failed"
                    }

                    Write-Host "All tools are available."
                '''
            }
        }

        stage('3. Install Dependencies') {
            steps {
                powershell '''
                    $python = 'C:\\Users\\hp\\expense_tracker\\.venv\\Scripts\\python.exe'

                    if (-not (Test-Path $python)) {
                        throw "Python executable not found: $python"
                    }

                    if (-not (Test-Path "requirements.txt")) {
                        throw "requirements.txt not found in Jenkins workspace"
                    }

                    Write-Host "Installing Python dependencies..."

                    & $python -m pip install -r requirements.txt

                    if ($LASTEXITCODE -ne 0) {
                        throw "Dependency installation failed"
                    }

                    Write-Host "Dependencies installed successfully."
                '''
            }
        }

        stage('4. Test Application') {
            steps {
                powershell '''
                    $python = 'C:\\Users\\hp\\expense_tracker\\.venv\\Scripts\\python.exe'

                    if (-not (Test-Path $python)) {
                        throw "Python executable not found: $python"
                    }

                    Write-Host "Testing Flask application..."

                    & $python -c "import app; print('Flask application loaded successfully')"

                    if ($LASTEXITCODE -ne 0) {
                        throw "Application test failed"
                    }

                    Write-Host "Application test passed."
                '''
            }
        }

        stage('5. Build Docker Image') {
            steps {
                powershell '''
                    $docker = 'C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'

                    if (-not (Test-Path $docker)) {
                        throw "Docker executable not found: $docker"
                    }

                    if (-not (Test-Path "Dockerfile")) {
                        throw "Dockerfile not found in Jenkins workspace"
                    }

                    Write-Host "Building Expense Tracker V1 image..."

                    & $docker build -t expense-tracker:v1 .

                    if ($LASTEXITCODE -ne 0) {
                        throw "Docker image build failed"
                    }

                    Write-Host "Verifying Docker image..."

                    & $docker image inspect expense-tracker:v1

                    if ($LASTEXITCODE -ne 0) {
                        throw "Docker image verification failed"
                    }

                    Write-Host "Docker image built successfully."
                '''
            }
        }

        stage('6. Deploy to Kubernetes') {
            steps {
                powershell '''
                    $kubectl = 'C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe'

                    if (-not (Test-Path $kubectl)) {
                        throw "kubectl executable not found: $kubectl"
                    }

                    if (-not (Test-Path "deployment.yaml")) {
                        throw "deployment.yaml not found in Jenkins workspace"
                    }

                    Write-Host "Checking Kubernetes context..."

                    & $kubectl config use-context docker-desktop

                    if ($LASTEXITCODE -ne 0) {
                        throw "Could not select docker-desktop context"
                    }

                    Write-Host "Applying Kubernetes configuration..."

                    & $kubectl apply -f deployment.yaml

                    if ($LASTEXITCODE -ne 0) {
                        throw "Kubernetes deployment failed"
                    }

                    Write-Host "Waiting for deployment rollout..."

                    & $kubectl rollout status deployment/expense-tracker --timeout=180s

                    if ($LASTEXITCODE -ne 0) {
                        throw "Kubernetes rollout failed"
                    }

                    Write-Host "Kubernetes deployment completed."
                '''
            }
        }

        stage('7. Verify Deployment') {
            steps {
                powershell '''
                    $kubectl = 'C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe'

                    if (-not (Test-Path $kubectl)) {
                        throw "kubectl executable not found: $kubectl"
                    }

                    Write-Host "Checking deployments..."

                    & $kubectl get deployments

                    if ($LASTEXITCODE -ne 0) {
                        throw "Could not list deployments"
                    }

                    Write-Host "Checking pods..."

                    & $kubectl get pods

                    if ($LASTEXITCODE -ne 0) {
                        throw "Could not list pods"
                    }

                    Write-Host "Checking services..."

                    & $kubectl get services

                    if ($LASTEXITCODE -ne 0) {
                        throw "Could not list services"
                    }

                    Write-Host "Deployment verification completed."
                '''
            }
        }
    }

    post {
        success {
            echo 'Expense Tracker V1 CI/CD pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check Console Output for the failed stage.'
        }
    }
}
