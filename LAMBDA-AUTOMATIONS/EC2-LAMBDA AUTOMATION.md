# Setting up lambda-EC2 automation with AWS EvENT Bridge
![lambda-eventBridge-ec2-deployment](https://github.com/user-attachments/assets/abee8455-fddb-4ba5-97ad-02f2faa17bca)

## project role/permission script
```xml

{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": [
				"ec2:RunInstances",
				"ec2:StopInstances",
				"ec2:DescribeInstances",
				"ec2:CreateTags"
			],
			"Resource": "*"
		}
	]
}
```


### Make sure your lambd configurations has the right permission to execute start and stop instances



## Python script to start instances
```xml 
import boto3

def lambda_handler(event, context):
    ec2 = boto3.client('ec2', region_name='us-east-1')
    
    user_data_script = '''#!/bin/bash #to install apache
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
'''

    # Launch instances
    response = ec2.run_instances(
        ImageId='ami',  # Make sure this is the correct AMI for your region
        InstanceType='t3.micro',
        KeyName='mykey', #replace with your own keypair
        MinCount=5,
        MaxCount=5,
        SecurityGroupIds=['sg-06c04602363a8a88f'],  # Ensure this is the ID, not the name (should be like 'sg-xxxxxxxx')
        SubnetId='subnet-id',
        UserData=user_data_script,  # Correct parameter name (case-sensitive)
        TagSpecifications=[{
            'ResourceType': 'instance',
            'Tags': [{'Key': 'AutoManaged', 'Value': 'True'}] #give your own key value pair{tags}
        }]
    )
    
    instance_ids = [inst['InstanceId'] for inst in response['Instances']]
    print(f"Launched instances: {instance_ids}")
```

## Deploy and create a test even to start instances
```xml
Give your test event a name and choose Hello world as Template
in the provided space below to input script insert this script

{}

then hit test to deploy
```

## Python scripts to Stop EC2 instances
```xml
import boto3

def lambda_handler(event, context):
    ec2 = boto3.client('ec2', region_name='us-east-1')

    instances = ec2.describe_instances(
        Filters=[
            {'Name': 'tag:AutoManaged', 'Values': ['True']},
            {'Name': 'instance-state-name', 'Values': ['running']}
        ]
    )

    instance_ids = [
        inst['InstanceId']
        for res in instances['Reservations']
        for inst in res['Instances']
    ]

    if instance_ids:
        ec2.stop_instances(InstanceIds=instance_ids)
        print(f"Stopped instances: {instance_ids}")
    else:
        print("No instances found to stop.")
```

## Deploy and create a test even to stop instances
```xml
Give your test event a name and choose Hello world as Template
in the provided space below to input script insert this script

{}

then hit test to deploy
```

## Create 2 AWS EventBridge to automate this deployment either Hourly,Daily,Weekly or Monthly
## START AND STOP Schedulers

## .A) START SCHEDULER STEPS
```xml
Amazon EventBridge Scheduler

Amazon EventBridge Scheduler is a serverless scheduler that allows you to create, run, and manage tasks from one central, managed service. Highly scalable, EventBridge Scheduler allows you to schedule millions of tasks that can invoke any AWS service as a target.

You will need to create 2 event schedulers
.1 "START scheduler" with cron-job to start your instances
.2 Select the TARGET SERVICE eg. "AWS LAMBDA Invoke"
.3 Choose the invoke function you created "START INSTANCES"
.4 Payload input this script "{}"
.5 Proceed with permission granting on the next page and create your scheduler.
```

### Repeat this same steps to create STOP SCHEDULER

