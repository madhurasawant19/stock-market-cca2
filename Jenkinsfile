
pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
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
                echo 'HTML validation passed.'
            }
        }

        stage('Build') {
            steps {
                echo 'Preparing StockMarket website.'
                archiveArtifacts artifacts: 'stockmarket.html',
                                 fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'StockMarket CI pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check the Jenkins Console Output.'
        }
    }
}
