pipeline {
    agent any

    environment {
        IMAGE_NAME = "backend-app"                       
        IMAGE_TAG = "v${BUILD_NUMBER}"                   
        FULL_IMAGE = "3.110.55.134:8081/${IMAGE_NAME}:${IMAGE_TAG}"

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
                    echo "Building Docker image: ${IMAGE_NAME}:${IMAGE_TAG}"
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push Docker Image to Nexus') {
            steps {
                script {
                    echo "Logging in and pushing to Nexus registry"
                    docker.withRegistry('http://3.110.55.134:8081', 'nexus-docker-credentials') {
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Clone Manifest Repo and Update Image Tag') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh """
                    rm -rf manifests
                    git clone --branch ${MANIFEST_REPO_BRANCH} https://$GITHUB_TOKEN@github.com/harinir2405/manifests.git
                    cd manifests

                    git config user.name "jenkins"
                    git config user.email "jenkins@ci.local"
                    git checkout -b update-image-${BUILD_NUMBER}

                    sed -i "s|image: .*|image: 3.110.55.134:8081/${IMAGE_NAME}:${IMAGE_TAG}|" deployment.yaml

                    git add deployment.yaml
                    git commit -m "Update image to 3.110.55.134:8081/${IMAGE_NAME}:${IMAGE_TAG}"
                    git push https://$GITHUB_TOKEN@github.com/harinir2405/manifests.git update-image-${BUILD_NUMBER}
                    """
                }
            }
        }
    }
}
