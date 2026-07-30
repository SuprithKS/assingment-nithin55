pipeline {
 
    agent any
 
    environment {
        IMAGE_NAME = "myapp"
        IMAGE_TAG = "${BUILD_NUMBER}"
        APP_PORT = "8081"
    }
 
    stages {
 
        stage('SCM Pull') {
            steps {
                checkout scm
            }
        }
 
        stage('Install Dependencies & Run Tests') {
            steps {
                sh 'mvn clean test'
            }
        }
 
        stage('Build Docker Image') {
            steps {
                sh """
                docker build \
                --cache-from ${IMAGE_NAME}:latest \
                -t ${IMAGE_NAME}:${IMAGE_TAG} \
                -t ${IMAGE_NAME}:latest .
                """
            }
        }
 
        stage('Deploy') {
            steps {
                sh """
                export IMAGE_TAG=${IMAGE_TAG}
 
                docker compose down || true
                docker compose up -d
                """
            }
        }
 
        stage('Readiness Check') {
            steps {
                sh '''
                echo "Waiting for application to become healthy..."
 
                for i in $(seq 1 30)
                do
                    if curl -s http://localhost:${APP_PORT}/actuator/health | grep -q '"status":"UP"'; then
                        echo "Application is UP."
                        exit 0
                    fi
 
                    echo "Attempt $i/30 - Application not ready yet..."
                    sleep 5
                done
 
                echo "Application failed to start."
 
                docker ps
                docker compose ps
                docker compose logs
 
                exit 1
                '''
            }
        }
    }
 
    post {
 
        success {
            echo "Build #${BUILD_NUMBER} completed successfully."
        }
 
        failure {
            echo "Pipeline Failed."
        }
 
        always {
            sh '''
            docker image prune -f
            docker container prune -f
            docker builder prune -f
            '''
 
            cleanWs()
        }
    }
}
