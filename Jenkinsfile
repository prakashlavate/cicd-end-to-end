pipeline {
    agent any

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_IMAGE = "pldockerhub7466/cicd-e2e"
    }

    stages {

        stage('Checkout App Source') {
            steps {
                git credentialsId: 'github-creds',
                    url: 'https://github.com/prakashlavate/cicd-end-to-end',
                    branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building Docker image..."
                    docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Checkout K8s Manifests') {
            steps {
                git credentialsId: 'github-creds',
                    url: 'https://github.com/prakashlavate/cicd-demo-manifests-repo/deploy.git',
                    branch: 'main'
            }
        }

        stage('Update Manifest and Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-creds',
                    usernameVariable: 'GIT_USERNAME',
                    passwordVariable: 'GIT_PASSWORD'
                )]) {
                    sh '''
                        echo "Updating image tag in deploy.yaml..."
                        git config user.name "jenkins-bot"
                        git config user.email "jenkins@example.com"

                        sed -i "s|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${IMAGE_TAG}|" deploy.yaml

                        git add deploy.yaml
                        git commit -m "Updated manifest with image tag ${IMAGE_TAG}"
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/prakashlavate/cicd-demo-manifests-repo/deploy.git HEAD:main
                    '''
                }
            }
        }

        // Optional: Trigger Argo CD Sync directly
        stage('Trigger Argo CD Sync') {
            steps {
                sh '''
                    argocd login localhost:8081 --username admin --password YOUR_ARGOCD_PASSWORD --insecure
                    argocd app sync your-argo-app-name
                '''
            }
        }

    }
}
