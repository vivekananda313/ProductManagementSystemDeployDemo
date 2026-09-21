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
                        credentialsId: 'dockerhub-jenkins-pat',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    bat '''
                        echo %DOCKER_PASSWORD% | docker login -u %DOCKER_USERNAME% --password-stdin

                        if %ERRORLEVEL% NEQ 0 (
                            echo Docker Hub login failed
                            exit /b 1
                        )

                        echo Docker Hub login successful

                        docker push %DOCKER_IMAGE%:%IMAGE_TAG%

                        if %ERRORLEVEL% NEQ 0 (
                            echo Docker image push failed
                            exit /b 1
                        )

                        echo Docker image pushed successfully
                    '''
                }
            }
        }

        // =========================
        // 6. Deploy to Kubernetes
        // =========================
        stage('Deploy to Kubernetes') {
            steps {

                bat 'kubectl set image deployment/product-management product-management=%DOCKER_IMAGE%:%IMAGE_TAG%'
            }
        }
    }

    // =========================
    // Post Actions
    // =========================
    post {

        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}
