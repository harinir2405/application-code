pipeline {
    agent any

    environment {
        IMAGE_NAME = "backend-app"                       // Name of the Docker image
        IMAGE_TAG = "v${BUILD_NUMBER}"                   // Tag using Jenkins build number
        FULL_IMAGE = "3.110.55.134:8081/${IMAGE_NAME}:${IMAGE_TAG}" // Nexus IP with correct port

        APP_REPO_URL = "https://github.com/harinir2405/application-code.git"
        MANIFEST_REPO_URL = "https://github.com/harinir2405/manifests.git"
        MANIFEST_REPO_BRANCH = "main"
    }

    stages {
        stage('Checkout App Repo') {
            steps {
                git url: "${APP_REPO_URL}", credentialsId: 'github-token', branch: 'main'
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
    withCredentials([usernamePassword(credentialsId: 'docker-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
        script {
            sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD} http://3.110.55.134:8081/repository/docker-hosted/"
            sh "docker push backend-app:v10"
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
                sed -i "s|image: .*|image: 3.110.55.134:8081/${IMAGE_NAME}:${IMAGE_TAG}|" deployment.yaml

                git add deployment.yaml
                git commit -m "Update image to 3.110.55.134:8081/${IMAGE_NAME}:${IMAGE_TAG}"
                git push https://$TOKEN@github.com/harinir2405/manifests.git update-image-$BUILD_NUMBER
                """
            }
        }
    }
}
