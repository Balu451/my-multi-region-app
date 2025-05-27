// Jenkinsfile

pipeline {
    agent any // This runs the pipeline on any available agent (your Jenkins server)

    environment {
        // AWS region where the primary application will be deployed
        AWS_REGION_PRIMARY = 'us-east-1'
        // AWS region where the secondary application will be deployed
        AWS_REGION_SECONDARY = 'us-west-2'
        // <<< IMPORTANT: Replace these with your actual S3 bucket names from main.tf >>>
        S3_BUCKET_PRIMARY = 'your-actual-primary-bucket-name'
        S3_BUCKET_SECONDARY = 'your-actual-secondary-bucket-name'
        // ID of the AWS credentials stored in Jenkins (from "Manage Credentials")
        AWS_CREDENTIALS_ID = 'aws-credentials-for-jenkins' // This should match the ID you gave to your AWS credentials in Jenkins
    }

    stages {
        // The 'Checkout Code' stage was removed in the previous step
        // because Jenkins handles cloning the repository automatically when you configure
        // the job to use "Pipeline script from SCM".

        stage('Build Application') {
            steps {
                echo "Building application..."
                // Example: Create dummy HTML files for demonstration
                sh 'echo "<h1>Hello from Primary Region!</h1>" > primary-index.html'
                sh 'echo "<h1>Hello from Secondary Region!</h1>" > secondary-index.html'
                // For a real application, you'd build your deployable artifact here
                // (e.g., mvn clean package, npm run build, etc.)
            }
        }

        stage('Deploy to Primary Region (${AWS_REGION_PRIMARY})') { // Use string interpolation for stage name
            steps {
                script {
                    // Use the AWS credentials and primary region defined in the environment block
                    withAWS(credentials: AWS_CREDENTIALS_ID, region: AWS_REGION_PRIMARY) {
                        echo "Deploying to Primary Region (${AWS_REGION_PRIMARY})..."
                        // Upload primary-index.html to the primary S3 bucket
                        // Removed --acl public-read because your S3 bucket does not allow ACLs
                        sh "aws s3 cp primary-index.html s3://${S3_BUCKET_PRIMARY}/index.html"
                        echo "Deployment to Primary S3 bucket successful."
                    }
                }
            }
        }

        stage('Deploy to Secondary Region (${AWS_REGION_SECONDARY})') { // Use string interpolation for stage name
            steps {
                script {
                    // Use the AWS credentials and secondary region defined in the environment block
                    withAWS(credentials: AWS_CREDENTIALS_ID, region: AWS_REGION_SECONDARY) {
                        echo "Deploying to Secondary Region (${AWS_REGION_SECONDARY})..."
                        // Upload secondary-index.html to the secondary S3 bucket
                        // Removed --acl public-read because your S3 bucket does not allow ACLs
                        sh "aws s3 cp secondary-index.html s3://${S3_BUCKET_SECONDARY}/index.html"
                        echo "Deployment to Secondary S3 bucket successful."
                    }
                }
            }
        }

        // --- Optional: Add a stage for DR/Failover Actions ---
        // This stage would only be triggered manually or when a failure is detected
        // For example, if you want to promote the RDS read replica to standalone DB
        // or update Route 53 records to point to the secondary after manual intervention.
        stage('Manual Failover Trigger (Optional)') {
            // This 'when' condition is an example; you could trigger this stage differently
            when { expression { return env.BRANCH_NAME == 'failover-trigger' } } // Example: triggered by pushing to a 'failover-trigger' branch
            steps {
                script {
                    echo "Initiating failover procedures (manual trigger)..."
                    // Add AWS CLI commands here to:
                    // 1. Promote secondary RDS read replica if needed
                    //    sh "aws rds promote-read-replica --db-instance-identifier secondary-db-replica --region ${AWS_REGION_SECONDARY}"
                    // 2. Potentially update Route 53 records (though failover routing policy handles this automatically with health checks)
                    //    This stage would be more for *preparatory* steps before Route 53 switches traffic
                    //    If Route 53 failover policy is already active, this stage is more for database promotion etc.
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
