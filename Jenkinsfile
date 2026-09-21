pipeline {

    agent any

    environment {

        // =========================
        // MySQL Configuration
        // =========================
        DB_URL = 'jdbc:mysql://localhost:3307/product_management_db'
        DB_PASSWORD = credentials('mysql-db-password')

        // =========================
        // Docker Hub Configuration
        // =========================
        DOCKER_IMAGE = 'vivek58254/product-management-system'
        IMAGE_TAG = "${BUILD_NUMBER}"

        // =========================
        // Kubernetes Configuration
        // =========================
        KUBECONFIG = 'C:\\Users\\VIVEKANANDA D\\.kube\\config'
    }

    options {
        skipDefaultCheckout(true)
    }

    stages {

        // =========================
        // 1. Checkout
        // =========================
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // =========================
        // 2. Build
        // =========================
        stage('Build') {
            steps {
                bat 'mvnw.cmd clean package -DskipTests'
            }
        }

        // =========================
        // 3. Test
        // =========================
        stage('Test') {
            steps {
                bat 'mvnw.cmd test'
            }
        }

        // =========================
        // 4. Docker Build
        // =========================
        stage('Docker Build') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE%:%IMAGE_TAG% .'
            }
        }

        // =========================
        // 5. Docker Push
        // =========================
        stage('Docker Push') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-login',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    powershell '''

                        Write-Host "======================================"
                        Write-Host "Logging into Docker Hub..."
                        Write-Host "Username: $env:DOCKER_USERNAME"
                        Write-Host "======================================"

                        $env:DOCKER_PASSWORD |
                            docker login `
                            --username $env:DOCKER_USERNAME `
                            --password-stdin

                        if ($LASTEXITCODE -ne 0) {
                            Write-Error "Docker Hub login failed"
                            exit 1
                        }

                        Write-Host "Docker Hub login successful"

                        Write-Host "======================================"
                        Write-Host "Pushing Docker Image..."
                        Write-Host "$env:DOCKER_IMAGE`:$env:IMAGE_TAG"
                        Write-Host "======================================"

                        docker push "$env:DOCKER_IMAGE`:$env:IMAGE_TAG"

                        if ($LASTEXITCODE -ne 0) {
                            Write-Error "Docker image push failed"
                            exit 1
                        }

                        Write-Host "Docker image pushed successfully"
                    '''
                }
            }
        }

        // =========================
        // 6. Deploy to Kubernetes
        // =========================
        stage('Deploy to Kubernetes') {
            steps {

                bat '''
                    echo ======================================
                    echo Deploying application to Kubernetes...
                    echo ======================================

                    kubectl set image deployment/product-management product-management=%DOCKER_IMAGE%:%IMAGE_TAG%

                    if %ERRORLEVEL% NEQ 0 (
                        echo Kubernetes deployment update failed
                        exit /b 1
                    )

                    echo Kubernetes deployment updated successfully
                '''
            }
        }
    }

    // =========================
    // Post Actions
    // =========================
    post {

        success {
            echo '======================================'
            echo 'PIPELINE COMPLETED SUCCESSFULLY!'
            echo '======================================'
        }

        failure {
            echo '======================================'
            echo 'PIPELINE FAILED!'
            echo '======================================'
        }
    }
}
