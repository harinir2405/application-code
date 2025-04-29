pipeline {
    agent any

    environment {
        IMAGE_NAME = "backend-app"                       // Name of the Docker image
        IMAGE_TAG = "v${BUILD_NUMBER}"                   // Tag using Jenkins build number
        FULL_IMAGE = "54.81.112.24:8081/${IMAGE_NAME}:${IMAGE_TAG}" // Nexus IP with correct port

        APP_REPO_URL = "https://github.com/farhanfist10/applicationcode.git"
        MANIFEST_REPO_URL = "https://github.com/farhanfist10/manifests.git"
        MANIFEST_REPO_BRANCH = "main"
    }

    stages {
        stage('Checkout App Repo') {
            steps {
                git url: "${APP_REPO_URL}", branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}") // Build the Docker image locally
                }
            }
        }

        stage('Push Docker Image to Nexus') {
            environment {
                DOCKER_CREDS = credentials('nexus-docker-credentials') // Jenkins credential ID
            }
            steps {
                script {
                    // Updated to use port 8081
                    docker.withRegistry('http://54.81.112.24:8081', 'nexus-docker-credentials') {
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Clone Manifest Repo and Update Image Tag') {
            environment {
                TOKEN = credentials('github-token') // GitHub token to push updates
            }
            steps {
                sh """
                rm -rf manifests
                git clone --branch $MANIFEST_REPO_BRANCH $MANIFEST_REPO_URL
                cd manifests
                git config user.name "jenkins"
                git config user.email "jenkins@ci.local"

                git checkout -b update-image-$BUILD_NUMBER

                # Update image URL with new port 8081
                sed -i "s|image: .*|image: 54.81.112.24:8081/${IMAGE_NAME}:${IMAGE_TAG}|" deployment.yaml

                git add deployment.yaml
                git commit -m "Update image to 54.81.112.24:8081/${IMAGE_NAME}:${IMAGE_TAG}"
                git push https://$TOKEN@github.com/farhanfist10/manifests.git update-image-$BUILD_NUMBER
                """
            }
        }
    }
}
