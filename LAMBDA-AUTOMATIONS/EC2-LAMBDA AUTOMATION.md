# Setting up lambda-EC2 automation with AWS EvENT Bridge
![lambda-eventBridge-ec2-deployment](https://github.com/user-attachments/assets/abee8455-fddb-4ba5-97ad-02f2faa17bca)

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

## Deploy and create a test even
```xml
Give your test event a name and choose Hello world as Template
in the provided space below to input script insert this script

{}

```

