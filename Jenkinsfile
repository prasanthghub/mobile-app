pipeline {
    agent any
    environment {
        REGISTRY = "your-docker-registry.com"
        IMAGE_NAME = "your-app"
        K8S_REPO = "git@github.com:yourorg/k8s-manifests.git"
        BRANCH = "main"
    }
    stages {
        stage('Checkout Source') {
            steps {
                git branch: "${BRANCH}", url: 'git@github.com:yourorg/your-app.git'
            }
        }
        stage('Quality Checks') {
            parallel {
                stage('Lint') { steps { sh 'mvn checkstyle:check' } }
                stage('Unit Tests') { steps { sh 'mvn test' } }
                stage('Integration Tests') { steps { sh 'mvn verify -Pintegration-tests' } }
            }
        }
        stage('Build & Push Docker Image') {
            steps {
                sh """
                    docker build -t $REGISTRY/$IMAGE_NAME:${BUILD_NUMBER} .
                    docker push $REGISTRY/$IMAGE_NAME:${BUILD_NUMBER}
                """
            }
        }
        stage('Update K8s Manifests') {
            steps {
                dir('k8s-manifests') {
                    git branch: "${BRANCH}", url: "${K8S_REPO}"
                    sh """
                        sed -i 's|image: .*$|image: $REGISTRY/$IMAGE_NAME:${BUILD_NUMBER}|g' deployment.yaml
                        git config user.name "jenkins-bot"
                        git config user.email "jenkins@example.com"
                        git add deployment.yaml
                        git commit -m "Update image to $REGISTRY/$IMAGE_NAME:${BUILD_NUMBER}"
                        git push origin ${BRANCH}
                    """
                }
            }
        }
    }
}
