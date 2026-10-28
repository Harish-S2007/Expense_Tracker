
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
                    Write-Host "Checking Python..."

                    $pythonCandidates = @(
                        "C:\\Users\\hp\\expense_tracker\\.venv\\Scripts\\python.exe",
                        "C:\\Users\\hp\\AppData\\Local\\Programs\\Python\\Python312\\python.exe",
                        "C:\\Program Files\\Python312\\python.exe"
                    )

                    $python = $null

                    foreach ($candidate in $pythonCandidates) {
                        if (Test-Path $candidate) {
                            $python = $candidate
                            break
                        }
                    }

                    if (-not $python) {
                        $commands = Get-Command python.exe -All -ErrorAction SilentlyContinue

                        foreach ($command in $commands) {
                            $candidate = $command.Source

                            if ($candidate -and
                                $candidate -notlike "*\\WindowsApps\\*") {
                                $python = $candidate
                                break
                            }
                        }
                    }

                    if (-not $python) {
                        throw "Python not found. Install Python and update the Jenkinsfile path."
                    }

                    Write-Host "Python path: $python"
                    & $python --version

                    if ($LASTEXITCODE -ne 0) {
                        throw "Python check failed"
                    }

                    $env:PYTHON_EXE = $python

                    Write-Host "Checking Docker..."

                    $dockerCandidates = @(
                        "C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe",
                        "C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe"
                    )

                    $docker = $null

                    foreach ($candidate in $dockerCandidates) {
                        if (Test-Path $candidate) {
                            $docker = $candidate
                            break
                        }
                    }

                    if (-not $docker) {
                        $command = Get-Command docker.exe -ErrorAction SilentlyContinue
                        if ($command) {
                            $docker = $command.Source
                        }
                    }

                    if (-not $docker) {
                        throw "Docker executable not found"
                    }

                    Write-Host "Docker path: $docker"
                    & $docker --version

                    if ($LASTEXITCODE -ne 0) {
                        throw "Docker check failed"
                    }

                    Write-Host "Checking kubectl..."

                    $kubectlCandidates = @(
                        "C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe",
                        "C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe"
                    )

                    $kubectl = $null

                    foreach ($candidate in $kubectlCandidates) {
                        if (Test-Path $candidate) {
                            $kubectl = $candidate
                            break
                        }
                    }

                    if (-not $kubectl) {
                        $command = Get-Command kubectl.exe -ErrorAction SilentlyContinue
                        if ($command) {
                            $kubectl = $command.Source
                        }
                    }

                    if (-not $kubectl) {
                        throw "kubectl executable not found"
                    }

                    Write-Host "kubectl path: $kubectl"
                    & $kubectl version --client

                    if ($LASTEXITCODE -ne 0) {
                        throw "kubectl check failed"
                    }
                '''
            }
        }

        stage('3. Install Dependencies') {
            steps {
                powershell '''
                    $python = $env:PYTHON_EXE

                    if (-not $python -or
                        -not (Test-Path $python)) {
                        throw "Python executable is not available"
                    }

                    if (-not (Test-Path "requirements.txt")) {
                        throw "requirements.txt not found in workspace"
                    }

                    Write-Host "Installing dependencies..."
                    & $python -m pip install -r requirements.txt

                    if ($LASTEXITCODE -ne 0) {
                        throw "Dependency installation failed"
                    }
                '''
            }
        }

        stage('4. Test Application') {
            steps {
                powershell '''
                    $python = $env:PYTHON_EXE

                    Write-Host "Testing Flask application..."
                    & $python -c "import app; print('Flask application loaded successfully')"

                    if ($LASTEXITCODE -ne 0) {
                        throw "Application test failed"
                    }
                '''
            }
        }

        stage('5. Build Docker Image') {
            steps {
                powershell '''
                    $docker = $null

                    $candidates = @(
                        "C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe",
                        "C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe"
                    )

                    foreach ($candidate in $candidates) {
                        if (Test-Path $candidate) {
                            $docker = $candidate
                            break
                        }
                    }

                    if (-not $docker) {
                        $command = Get-Command docker.exe -ErrorAction SilentlyContinue
                        if ($command) {
                            $docker = $command.Source
                        }
                    }

                    if (-not $docker) {
                        throw "Docker executable not found"
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
                '''
            }
        }

        stage('6. Deploy to Kubernetes') {
            steps {
                powershell '''
                    $kubectl = $null

                    $candidates = @(
                        "C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe",
                        "C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe"
                    )

                    foreach ($candidate in $candidates) {
                        if (Test-Path $candidate) {
                            $kubectl = $candidate
                            break
                        }
                    }

                    if (-not $kubectl) {
                        $command = Get-Command kubectl.exe -ErrorAction SilentlyContinue
                        if ($command) {
                            $kubectl = $command.Source
                        }
                    }

                    if (-not $kubectl) {
                        throw "kubectl executable not found"
                    }

                    Write-Host "Checking Kubernetes context..."
                    & $kubectl config use-context docker-desktop

                    if ($LASTEXITCODE -ne 0) {
                        throw "Could not select docker-desktop context"
                    }

                    if (-not (Test-Path "deployment.yaml")) {
                        throw "deployment.yaml not found in workspace"
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
                '''
            }
        }

        stage('7. Verify Deployment') {
            steps {
                powershell '''
                    $kubectl = $null

                    $candidates = @(
                        "C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe",
                        "C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe"
                    )

                    foreach ($candidate in $candidates) {
                        if (Test-Path $candidate) {
                            $kubectl = $candidate
                            break
                        }
                    }

                    if (-not $kubectl) {
                        $command = Get-Command kubectl.exe -ErrorAction SilentlyContinue
                        if ($command) {
                            $kubectl = $command.Source
                        }
                    }

                    if (-not $kubectl) {
                        throw "kubectl executable not found"
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
