pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "yourdockerhub/demo-app"
        DOCKER_TAG = "${BUILD_NUMBER}"
        DOCKER_CREDENTIALS = 'docker-hub-credentials'
        KUBECONFIG = credentials('kubeconfig')
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_CREDENTIALS) {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push('latest')
                    }
                }
            }
        }
        
        stage('Update Helm Values') {
            steps {
                script {
                    sh """
                        cd helm/demo-app
                        sed -i 's|tag:.*|tag: "${DOCKER_TAG}"|' values.yaml
                    """
                    
                    // Commit and push the updated values file to trigger ArgoCD
                    sh """
                        git config --global user.email "jenkins@example.com"
                        git config --global user.name "Jenkins"
                        git add helm/demo-app/values.yaml
                        git commit -m "Update image tag to ${DOCKER_TAG}"
                        git push origin main
                    """
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}