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


    // Prevent Jenkins automatic checkout
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
        // 4. Docker Login
        // =========================
        stage('Docker Login') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-jenkins-pat',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    powershell '''
                        Write-Host "Docker login started..."

                        $env:DOCKER_PASSWORD |
                            docker login `
                            --username $env:DOCKER_USERNAME `
                            --password-stdin

                        if ($LASTEXITCODE -ne 0) {
                            Write-Error "Docker Hub login failed."
                            exit 1
                        }

                        Write-Host "Docker login successful."
                    '''
                }
            }
        }


        // =========================
        // 5. Docker Build
        // =========================
        stage('Docker Build') {
            steps {

                bat 'docker build -t %DOCKER_IMAGE%:%IMAGE_TAG% .'
            }
        }


        // =========================
        // 6. Docker Push
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

                    bat 'docker push %DOCKER_IMAGE%:%IMAGE_TAG%'
                }
            }
        }


        // =========================
        // 7. Deploy to Kubernetes
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
