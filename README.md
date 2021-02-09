# AWS-CodeDeploy

__AWS CodeDeploy vs. AWS CodePipeline:__ AWS CodeDeploy is more of the Deployment As A Service (DAAS) wherein once your code is ready, you can use CodeDeploy to automate the process of deploying your code across EC2 instances and other resources. Whereas AWS CodePipeline is more of a *Continuous Delivery* type of service which automates code deplpoyments to a number of resources including even on-premise servers. Pipeline deploys code EVERY TIME that it changes. Also, you can configure several different kind of deployments such as in staging, production etc.

Create a simple pipeline that uses CodeDeploy to deploy a sample application from an Amazon S3 bucket to Amazon EC2 instances running Amazon Linux. After the two-stage pipeline is created, I will add a third stage.

1. The recommended approach is to upload your files as a zip file so that there are no issues downstream.
2. Any AWS resources that you are going to use in your pipeline must be created in the SAME region as CodePipeline.

__Step 1: CREATE AN S3 BUCKET FOR YOUR APPLICATION__
1. Create an AWS Bucket for your pipeline, we will call it aws-bucket-for-code-pipeline (remember, no upperclase alphabets)
2: Go to the properties and enable versioning on bucket (versioning preserves all versions of objects and helps recover from accidental deletions)
3: Upload Sample_App_Windows.zip to the bucket (remember to always keep your application zipped)

__Step 2: CREATE AN EC2 Windows Instance and Install CodeDeploy on it__
CodeDeploy is a software package that allows that instance to be used in CodeDeploy deployments

1. Before creating the instance, we must create a __Role__ that will allow appropriate access before and during deployment (role = temporary permission)
2. So we go to IAM, create a role for AWS Services specifying that we want to allow EC2 instance to call other AWS services and name the role __EC2InstanceRole__
3. Now we will launch an instance so we go to EC2 then launch an instance by selecting the __Microsoft Windows Server 2019 Base with Containers__ AMI
4. In configure instances, select to instantiate 2 instances and select auto-assign IP enabled and select EC2Instance role that we previously created. You also need to specify the following script in the advanced section and put the following text:

*<powershell>  
New-Item -Path c:\temp -ItemType "directory" -Force
powershell.exe -Command Read-S3Object -BucketName bucket-name/latest -Key codedeploy-agent.msi -File c:\temp\codedeploy-agent.msi
Start-Process -Wait -FilePath c:\temp\codedeploy-agent.msi -WindowStyle Hidden
</powershell>*

where the bucket-name is the name of the bucket specific to that region. This script basically installs CodeDeploy agent on our instance. For our case, since our instance is hosted in the North Virginia (US-East) region, therefore the bucket name is: __aws-codedeploy-us-east-1__

5. In the tags section, add the Name tag, SimpleCodePipelineDemo
6. In the next step, configure the Security Groups to open port 80 so that the public endpoint of your instance is reachable
7. Generate a new key pair and download the private key pair and store it in a safe and secure location

__Step 3: CREATE an application in CodeDeploy__
In CodeDeploy, an application is just a name or an identifier that you use to point to specific code







