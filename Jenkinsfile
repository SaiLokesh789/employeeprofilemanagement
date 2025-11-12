pipeline {
    agent any

    environment {
        // Uncomment below if pushing to DockerHub
        // DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        DOCKER_IMAGE = "employeeprofilemanagement_image"
        CONTAINER_NAME = "employeeprofilemanagement"

        // Database URL used when dockerization is done for the application
        DB_URL = "jdbc:postgresql://host.docker.internal:5432/epms_db"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/SaiLokesh789/employeeprofilemanagement'
            }
        }

        stage('Build') {
            steps {
                sh '''
                    echo "🚀 Building Docker image..."
                    docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} -f Dockerfile .
                '''
            }
        }

        stage('Run Container') {
            steps {
                script {
                    sh '''
                        echo "🧹 Stopping and removing old container if exists..."
                        docker ps -a --filter "name=${CONTAINER_NAME}" --format "{{.ID}}" | xargs -r docker rm -f
                        
                        echo "🐳 Running new Docker container..."
                        docker run -d \
                            --name ${CONTAINER_NAME} \
                            -p 8200:8200 \
                            -e DB_URL=${DB_URL} \
                            ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Checkout, Build, and Container Run completed successfully!"
        }
        failure {
            echo "❌ Build or Deployment failed!"
        }
    }
}
