# AWS-CodePipeline

Create a simple pipeline that uses CodeDeploy to deploy a sample application from an Amazon S3 bucket to Amazon EC2 instances running Amazon Linux. After the two-stage pipeline is created, I will add a third stage.

1. The recommended approach is to upload your files as a zip file so that there are no issues downstream.
2. Any AWS resources that you are going to use in your pipeline must be created in the SAME region as CodePipeline.

Step 1: Create an AWS Bucket for your pipeline, we will call it aws-bucket-for-code-pipeline (remember, no upperclase alphabets)
Step 2: Go to the properties and enable versioning on bucket (versioning preserves all versions of objects and helps recover from accidental deletions)
