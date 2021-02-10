# AWS-Creating A Simple Pipeline

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
So we go to IAM, create a role for AWS Services specifying that we want to allow EC2 instance to call other AWS services and name the role __*EC2InstanceRole*__ and attach the policy titled __*AmazonEC2RoleforAWSCodeDeploy*__
2. Now we will launch an instance so we go to EC2 then launch an instance by selecting the __*Microsoft Windows Server 2019 Base with Containers*__ AMI
3. In configure instances, select to instantiate 2 instances and select auto-assign IP enabled and select EC2Instance role that we previously created. You also need to specify the following script in the advanced section and put the following text:

*<powershell>  
New-Item -Path c:\temp -ItemType "directory" -Force
powershell.exe -Command Read-S3Object -BucketName bucket-name/latest -Key codedeploy-agent.msi -File c:\temp\codedeploy-agent.msi
Start-Process -Wait -FilePath c:\temp\codedeploy-agent.msi -WindowStyle Hidden
</powershell>*

where the bucket-name is the name of the bucket specific to that region. This script basically installs CodeDeploy agent on our instance. For our case, since our instance is hosted in the North Virginia (US-East) region, therefore the bucket name is: __*aws-codedeploy-us-east-1*__

4. In the tags section, add the Name tag, SimpleCodePipelineDemo
5. In the next step, configure the Security Groups to open port 80 so that the public endpoint of your instance is reachable
6. Generate a new key pair and download the private key pair and store it in a safe and secure location

__Step 3: CREATE an application in CodeDeploy__

In CodeDeploy, an application is just a name or an identifier that you use to point to specific code.

1. Go to CodeDeploy -> applications and create a new application. You can name it SimpleCodeDeployDemo and select EC2/On-Premises as computer platform
2. Create a deployment group for your application and name it SimpleDemoDeploymentGroup
3. But you need to create/assign a service role to this deployment group so that the CodeDeploy service can access other AWS resources when required. For this, you must create a new role and select CodeDeploy in the list of given services, so an appropriate policy would be attached to the role, now crete and copy the ARN and then paste it in the CodeDeployment role tab
4. Select In-Place as the deployment type (this redeploys on the same instance with a small downtime). The other option is Blue/Green in which case new instances are created and once code is deployed on them and they are registered with the load-balancer, then the old instances are deregistered and can be deleted.
5. Select EC2 instances as the deployment target and add the same tags that you used when creating the two instances to correctly identify the correct instances
6. In Deployment type: OneAtATime deployment mode

At this point, we need to create a Load Balancer which will basically ensure that load-balancing is done and more importantly, that the traffic to instances is temporarily blocked to instances on which code is being deployed and redirected to ones on which it is not and then redirected again to the first one once the deployment is complete and finally, once all code updates are done, the traffic is dynamically balanced again between the two (or X) instances.

Go to Load Balancing and create a new Application Load Balancer (ALB). Remember, you need to specify at least two subnets in two different AZs (i.e. two Availability Zones because each displayed subnet for selection is in a different availability zone) so that is one AZ goes down, at least the ALB can keep on working in the other one i.e. reliability. Select IPv4 as address type. NOTE: if you selected an HTTPS listener (port 443 for secure communication), you MUST also specify a certificate (perhaps from AWS Certificate Manager), otherwise for HTTP it is OK.

8. At the last step of creating an ALB, you need to specify a security group because the ALB needs to communicate with instances on specified port for communication as well as on another port for health check. Just create a new Security Group which will automatically contain a default rule (for port 80 usually)

9. You need to configure routing i.e. whether it is for instances (by ID or IP) or AWS Lambda etc. 

10. The next screen simply shows the instances, so you can easily select them.

11. Now go back to the deployment screen and select the ALB and then create the deployment.

__Step 4: Create your First Pipeline in AWS CodeDeploy__

1. Open CodePipeline from the AWS management console
2. Create a new pipeline and name it __SimpleCodeDeployDemo-CodePipeline__
3. You need to create a new service role to allow for the pipeline to access AWS resources during the automated deployment. The role name and policy are already given by default and are in the following format: __AWSCodePipelineServiceRole-us-west-2-SimpleCodeDeployDemo-SimpleCodeDeployDemo-CodePipeline__ that is:

AWSCodePipelineServiceRole-region-pipelineName

4. In the next phase, you need to specify the source for the pipeline. In our case, it is AWS S3, so we select the correct bucket after selecting S3
5. The __*Object Key*__ is the name of the zipped file which contains the application including the extension
6. Next screen allows you to configure how to detect changes to your source i.e. app in S3, default is AWS CloudWatch Events which is fine
7. In the __Build Stage__ settings, skip it
8. In the __Deploy Provider__, select __AWS CodeDeploy__
9. Selection region (defaults to this one), select application and select the deployment group

Results, Well the deployment failed because the bucket was not located in us-west region









