pipeline {

    agent any

    environment {

        // MySQL
        DB_URL = 'jdbc:mysql://localhost:3307/product_management_db'
        DB_PASSWORD = credentials('mysql-db-password')

        // Docker Hub
        DOCKER_IMAGE = 'vivek58254/product-management-system'
        IMAGE_TAG = "${BUILD_NUMBER}"

        // Kubernetes
        KUBECONFIG = 'C:\\Users\\VIVEKANANDA D\\.kube\\config'
    }

    options {
        skipDefaultCheckout(true)
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                bat 'mvnw.cmd clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                bat 'mvnw.cmd test'
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE%:%IMAGE_TAG% .'
            }
        }

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
                        echo Logging into Docker Hub...

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

        stage('Deploy to Kubernetes') {
            steps {
                bat 'kubectl set image deployment/product-management product-management=%DOCKER_IMAGE%:%IMAGE_TAG%'
            }
        }
    }

    post {

        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}
