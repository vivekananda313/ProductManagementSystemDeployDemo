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
        // 5. Docker Credential Test
        // =========================
        stage('Docker Credential Test') {
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
                        Write-Host "Docker Hub Credential Test"
                        Write-Host "======================================"

                        Write-Host "Jenkins username: $env:DOCKER_USERNAME"

                        $bytes = [System.Text.Encoding]::UTF8.GetBytes(
                            $env:DOCKER_PASSWORD
                        )

                        $sha256 = [System.Security.Cryptography.SHA256]::Create()

                        $hash = [BitConverter]::ToString(
                            $sha256.ComputeHash($bytes)
                        ).Replace("-", "")

                        Write-Host "Jenkins credential SHA256: $hash"

                        Write-Host "======================================"
                        Write-Host "Credential test completed"
                        Write-Host "======================================"
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
                    echo Deploying application to Kubernetes...

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
            echo 'Pipeline completed successfully!'
            echo '======================================'
        }

        failure {
            echo '======================================'
            echo 'Pipeline failed!'
            echo '======================================'
        }
    }
}
