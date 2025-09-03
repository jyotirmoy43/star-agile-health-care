

pipeline {
    agent any

    tools {
        maven '3.6.3'
    }

    environment {
        MAVEN_OPTS = "-Dmaven.repo.local=.m2/repository"
        IMAGE_NAME = "jyotirmoy43/myimg1"
        APP_PORT = "8092"
        K8S_NAMESPACE = "default"
        DEPLOYMENT_NAME = "health-care-app"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo '📥 Checking out code from GitHub...'
                git url: 'https://github.com/jyotirmoy43/star-agile-health-care.git'
            }
        }

        stage('Build & Test') {
            steps {
                echo '🔨 Compiling and testing the code...'
                sh 'mvn clean package -DskipTests=false'
            }
        }

        stage('Code Quality - Checkstyle') {
            steps {
                echo '🧹 Running Checkstyle...'
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image...'
                sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                echo '📤 Pushing Docker image to Docker Hub...'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_NAME:$BUILD_NUMBER
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo '🚀 Deploying application to Kubernetes...'
                sh '''
                    # Make a copy of deployment.yaml and inject the new image
                    cp k8s/deployment.yaml k8s/deployment-temp.yaml
                    sed -i "s|IMAGE_PLACEHOLDER|$IMAGE_NAME:$BUILD_NUMBER|g" k8s/deployment-temp.yaml

                    # Apply manifests
                    kubectl apply -f k8s/deployment-temp.yaml -n $K8S_NAMESPACE
                    kubectl apply -f k8s/service.yaml -n $K8S_NAMESPACE

                    # Wait for rollout
                    kubectl rollout status deployment/$DEPLOYMENT_NAME -n $K8S_NAMESPACE
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Build and Kubernetes deployment completed successfully!'
            echo "🌐 App should be available via Kubernetes Service on port ${APP_PORT}"
        }
        failure {
            echo '❌ Build or deployment failed. Check the logs.'
        }
    }
}
