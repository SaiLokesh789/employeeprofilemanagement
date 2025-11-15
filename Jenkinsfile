pipeline {
    agent any

    environment {

        KUBECONFIG = "/Users/sailokesh/.kube/config"
        // Docker image name
        DOCKER_IMAGE = "employeeprofilemanagement_image"

        // PostgreSQL database URL (change if needed)
        DB_URL = "jdbc:postgresql://host.docker.internal:5432/epms_db"
        DB_USERNAME = "postgres"
        DB_PASSWORD = "postgres"

        // Kubernetes Namespace
        K8S_NAMESPACE = "epms-namespace"
    }

    stages {

        /* -----------------------------------------
           1. CHECKOUT CODE
        ----------------------------------------- */
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/SaiLokesh789/employeeprofilemanagement'
            }
        }

        /* -----------------------------------------
           2. BUILD DOCKER IMAGE
        ----------------------------------------- */
        stage('Build') {
            steps {
                sh '''
                    echo "🚀 Building Docker image..."
                    docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} -f Dockerfile .
                '''
            }
        }

        /* -----------------------------------------
           3. RUN DOCKER LOCALLY (optional)
        ----------------------------------------- */
        // stage('Run Container') {
        //     steps {
        //         script {
        //             sh '''
        //                 echo "🧹 Cleaning up old container..."

        //                 if [ "$(docker ps -aq -f name=employeeprofilemanagement)" ]; then
        //                     docker stop employeeprofilemanagement || true
        //                     docker rm -f employeeprofilemanagement || true
        //                 fi

        //                 echo "🐳 Running container..."
        //                 docker run -d \
        //                     --name employeeprofilemanagement \
        //                     -e SPRING_DATASOURCE_URL=${DB_URL} \
        //                     -e SPRING_DATASOURCE_USERNAME=${DB_USERNAME} \
        //                     -e SPRING_DATASOURCE_PASSWORD=${DB_PASSWORD} \
        //                     -p 8200:8200 \
        //                     ${DOCKER_IMAGE}:${BUILD_NUMBER}
        //             '''
        //         }
        //     }
        // }

        /* -----------------------------------------
           4. KUBERNETES DEPLOYMENT
        ----------------------------------------- */
        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    echo "📦 Updating Docker image in Kubernetes manifest..."

                    # macOS-compatible sed fix
                    sed -i '' "s|image: .*|image: ${DOCKER_IMAGE}:${BUILD_NUMBER}|g" deployment.yaml

                    echo "📁 Applying Namespace..."
                    kubectl apply -f namespace.yaml

                    echo "🚀 Deploying to Kubernetes..."
                    kubectl apply -n employeemanagementsystem -f deployment.yaml
                    kubectl apply -n employeemanagementsystem -f service.yaml

                    echo "⏳ Waiting for rollout to finish..."
                    kubectl rollout status deployment/employeemanagementsystem-deployment -n employeemanagementsystem

                    echo "✅ Kubernetes deployment completed!"
                '''
            }
        }
    }

    post {
        success {
            echo "🎉 SUCCESS: Checkout, Build, Docker, and Kubernetes Deployment Completed!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}
