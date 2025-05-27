// Jenkinsfile

pipeline {
    agent any

    environment {
        AWS_REGION_PRIMARY = 'us-east-1'
        AWS_REGION_SECONDARY = 'us-west-2'
        // <<< UPDATE THESE LINES WITH THE ACTUAL BUCKET NAMES FROM YOUR main.tf >>>
        S3_BUCKET_PRIMARY = 'your-unique-primary-bucket-name-12345' // <-- REPLACE THIS
        S3_BUCKET_SECONDARY = 'your-unique-primary-bucket-name-67890' // <-- REPLACE THIS
        AWS_CREDENTIALS_ID = 'aws-credentials-for-jenkins'
    }

    stages {
        stage('Build Application') {
            steps {
                echo "Building application..."
                sh 'echo "<h1>Hello from Primary Region!</h1>" > primary-index.html'
                sh 'echo "<h1>Hello from Secondary Region!</h1>" > secondary-index.html'
            }
        }

        stage('Deploy to Primary Region (${AWS_REGION_PRIMARY})') {
            steps {
                script {
                    withAWS(credentials: AWS_CREDENTIALS_ID, region: AWS_REGION_PRIMARY) {
                        echo "Deploying to Primary Region (${AWS_REGION_PRIMARY})..."
                        sh "aws s3 cp primary-index.html s3://${S3_BUCKET_PRIMARY}/index.html"
                        echo "Deployment to Primary S3 bucket successful."
                    }
                }
            }
        }

        stage('Deploy to Secondary Region (${AWS_REGION_SECONDARY})') {
            steps {
                script {
                    withAWS(credentials: AWS_CREDENTIALS_ID, region: AWS_REGION_SECONDARY) {
                        echo "Deploying to Secondary Region (${AWS_REGION_SECONDARY})..."
                        sh "aws s3 cp secondary-index.html s3://${S3_BUCKET_SECONDARY}/index.html"
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
