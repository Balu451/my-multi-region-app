// Jenkinsfile

pipeline {
    agent any

    environment {
        AWS_REGION_PRIMARY = 'us-east-1'
        AWS_REGION_SECONDARY = 'us-west-2'
        S3_BUCKET_PRIMARY = 'your-unique-primary-bucket-name-12345' // <<< IMPORTANT: Use your actual S3 bucket name from main.tf
        S3_BUCKET_SECONDARY = 'your-unique-secondary-bucket-name-67890' // <<< IMPORTANT: Use your actual S3 bucket name from main.tf
        AWS_CREDENTIALS_ID = 'aws-credentials-for-jenkins' // This should match the ID you gave to your AWS credentials in Jenkins
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Ensure this URL matches your actual GitHub repository
                git branch: 'main', url: 'https://github.com/your-github-username/your-repository-name.git' // <<< IMPORTANT: Update with your repo details
            }
        }

        stage('Build Application') {
            steps {
                echo "Building application..."
                sh 'echo "<h1>Hello from Primary Region!</h1>" > primary-index.html'
                sh 'echo "<h1>Hello from Secondary Region!</h1>" > secondary-index.html'
            }
        }

        stage('Deploy to Primary Region (us-east-1)') {
            steps {
                script {
                    withAWS(credentials: AWS_CREDENTIALS_ID, region: AWS_REGION_PRIMARY) {
                        echo "Deploying to Primary Region (${AWS_REGION_PRIMARY})..."
                        sh "aws s3 cp primary-index.html s3://${S3_BUCKET_PRIMARY}/index.html --acl public-read"
                        echo "Deployment to Primary S3 bucket successful."
                    }
                }
            }
        }

        stage('Deploy to Secondary Region (us-west-2)') {
            steps {
                script {
                    withAWS(credentials: AWS_CREDENTIALS_ID, region: AWS_REGION_SECONDARY) {
                        echo "Deploying to Secondary Region (${AWS_REGION_SECONDARY})..."
                        sh "aws s3 cp secondary-index.html s3://${S3_BUCKET_SECONDARY}/index.html --acl public-read"
                        echo "Deployment to Secondary S3 bucket successful."
                    }
                }
            }
        }

        stage('Manual Failover Trigger (Optional)') {
            when { expression { return env.BRANCH_NAME == 'failover-trigger' } }
            steps {
                script {
                    echo "Initiating failover procedures (manual trigger)..."
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed! Check logs for details."
        }
    }
}