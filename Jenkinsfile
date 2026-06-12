pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "harjotsingh2k19"
        GITOPS_REPO    = "https://github.com/HarjotSingh2k19/retail-store-gitops.git"
        IMAGE_TAG      = "${BUILD_NUMBER}"
    }

    stages {

        stage('1 - Checkout') {
            steps {
                echo "Checking out source code..."
                checkout scm
            }
        }

        stage('2 - Build Images') {
            steps {
                echo "Building all 5 images with tag: ${IMAGE_TAG}"
                sh "docker build -t ${DOCKERHUB_USER}/retail-store-ui:${IMAGE_TAG} src/ui/"
                sh "docker build -t ${DOCKERHUB_USER}/retail-store-catalog:${IMAGE_TAG} src/catalog/"
                sh "docker build -t ${DOCKERHUB_USER}/retail-store-cart:${IMAGE_TAG} src/cart/"
                sh "docker build -t ${DOCKERHUB_USER}/retail-store-orders:${IMAGE_TAG} src/orders/"
                sh "docker build -t ${DOCKERHUB_USER}/retail-store-checkout:${IMAGE_TAG} src/checkout/"
            }
        }

        stage('3 - Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'DOCKERHUB_CREDS',
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_PASS'
                )]) {
                    sh 'echo $DH_PASS | docker login -u $DH_USER --password-stdin'
                    sh "docker push ${DOCKERHUB_USER}/retail-store-ui:${IMAGE_TAG}"
                    sh "docker push ${DOCKERHUB_USER}/retail-store-catalog:${IMAGE_TAG}"
                    sh "docker push ${DOCKERHUB_USER}/retail-store-cart:${IMAGE_TAG}"
                    sh "docker push ${DOCKERHUB_USER}/retail-store-orders:${IMAGE_TAG}"
                    sh "docker push ${DOCKERHUB_USER}/retail-store-checkout:${IMAGE_TAG}"
                }
            }
        }

        stage('4 - Update GitOps Repo') {
            steps {
                withCredentials([string(
                    credentialsId: 'GITHUB_TOKEN',
                    variable: 'GH_TOKEN'
                )]) {
                    sh """
                        rm -rf retail-store-gitops
                        git clone https://\$GH_TOKEN@github.com/HarjotSingh2k19/retail-store-gitops.git
                        cd retail-store-gitops
                        sed -i 's|tag: ".*"|tag: "${IMAGE_TAG}"|g' helm/values.yaml
                        git config user.email "jenkins@ci.local"
                        git config user.name "Jenkins CI"
                        git add helm/values.yaml
                        git commit -m "ci: bump all image tags to ${IMAGE_TAG} [skip ci]"
                        git push
                    """
                }
            }
        }

        stage('5 - Done') {
            steps {
                echo "All images pushed with tag: ${IMAGE_TAG}"
                echo "ArgoCD will detect the change and deploy within 3 minutes."
            }
        }
    }

    post {
        success {
            echo "Pipeline SUCCESS — build ${IMAGE_TAG} deployed via GitOps"
        }
        failure {
            echo "Pipeline FAILED — check stage logs above"
        }
        always {
            sh 'docker logout || true'
        }
    }
}
