
pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        DOCKER_USER = 'madhuraaa19'
        IMAGE_NAME = 'stockmarket-website'
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'Source code checked out successfully.'
            }
        }

        stage('Validate HTML') {
            steps {
                sh '''
                    test -f stockmarket.html
                    grep -qi "<html" stockmarket.html
                    grep -qi "</html>" stockmarket.html
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${DOCKER_USER}/${IMAGE_NAME}:${BUILD_NUMBER} .
                    docker tag ${DOCKER_USER}/${IMAGE_NAME}:${BUILD_NUMBER} ${DOCKER_USER}/${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_TOKEN'
                )]) {
                    sh '''
                        echo "$DH_TOKEN" | docker login -u "$DH_USER" --password-stdin
                        docker push ${DOCKER_USER}/${IMAGE_NAME}:${BUILD_NUMBER}
                        docker push ${DOCKER_USER}/${IMAGE_NAME}:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                sh '''
                    kubectl --kubeconfig="$KUBECONFIG" apply -f k8s/
                    kubectl --kubeconfig="$KUBECONFIG" set image deployment/stockmarket stockmarket=${DOCKER_USER}/${IMAGE_NAME}:${BUILD_NUMBER}
                    kubectl --kubeconfig="$KUBECONFIG" rollout status deployment/stockmarket --timeout=120s
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl --kubeconfig="$KUBECONFIG" get deployments
                    kubectl --kubeconfig="$KUBECONFIG" get pods
                    kubectl --kubeconfig="$KUBECONFIG" get services
                '''
            }
        }
    }

    post {
        success {
            echo 'StockMarket CI/CD pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
