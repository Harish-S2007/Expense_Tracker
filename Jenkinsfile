
pipeline {
    agent any

    options {
        timestamps()
    }

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

                    if (!(Test-Path $python)) {
                        throw "Python not found: $python"
                    }

                    if (!(Test-Path $docker)) {
                        throw "Docker not found: $docker"
                    }

                    if (!(Test-Path $kubectl)) {
                        throw "kubectl not found: $kubectl"
                    }

                    & $python --version
                    if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }

                    & $docker --version
                    if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }

                    & $kubectl version --client
                    if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }
                '''
            }
        }

        stage('3. Install Dependencies') {
            steps {
                powershell '''
                    $python = 'C:\\Users\\hp\\expense_tracker\\.venv\\Scripts\\python.exe'

                    & $python -m pip install -r requirements.txt

                    if ($LASTEXITCODE -ne 0) {
                        exit $LASTEXITCODE
                    }
                '''
            }
        }

        stage('4. Test Application') {
            steps {
                powershell '''
                    $python = 'C:\\Users\\hp\\expense_tracker\\.venv\\Scripts\\python.exe'

                    & $python -c "import app; print('Flask application loaded successfully')"

                    if ($LASTEXITCODE -ne 0) {
                        exit $LASTEXITCODE
                    }
                '''
            }
        }

        stage('5. Build Docker Image') {
            steps {
                powershell '''
                    $docker = 'C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'

                    & $docker build -t expense-tracker:v1 .

                    if ($LASTEXITCODE -ne 0) {
                        exit $LASTEXITCODE
                    }

                    & $docker image inspect expense-tracker:v1

                    if ($LASTEXITCODE -ne 0) {
                        exit $LASTEXITCODE
                    }
                '''
            }
        }

        stage('6. Deploy to Kubernetes') {
            steps {
                powershell '''
                    $kubectl = 'C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe'

                    & $kubectl config use-context docker-desktop

                    if ($LASTEXITCODE -ne 0) {
                        exit $LASTEXITCODE
                    }

                    & $kubectl apply -f deployment.yaml

                    if ($LASTEXITCODE -ne 0) {
                        exit $LASTEXITCODE
                    }

                    & $kubectl rollout status deployment/expense-tracker --timeout=180s

                    if ($LASTEXITCODE -ne 0) {
                        exit $LASTEXITCODE
                    }
                '''
            }
        }

        stage('7. Verify Deployment') {
            steps {
                powershell '''
                    $kubectl = 'C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe'

                    & $kubectl get deployments
                    if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }

                    & $kubectl get pods
                    if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }

                    & $kubectl get services
                    if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }
                '''
            }
        }
    }

    post {
        success {
            echo 'Expense Tracker V1 CI/CD pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Open Console Output to identify the failed stage.'
        }
    }
}
