
This file is created to deploy same infrastructure using AWS `CLI commands`. 

- First, We will install AWS CLI v2. 

```bash
sudo yum update -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

- Write credentials using the command below:
```bash
aws configure
```
-Run the command below to check whether it is okay or not:
```bash
aws sts get-caller-identity --query Account --output text
```
1. Create Security Group
```bash
aws ec2 create-security-group --group-name umut-roman-numbers-converter-sec-grp --description "This is security group is created to open ssh and http connection from anywhere"
```
- To check the security group, run the command below:
```bash
aws ec2 describe-security-groups --group-names umut-roman-numbers-converter-sec-grp
```

2. Create rules of the security group:
```bash
aws ec2 authorize-security-group-ingress --group-names umut-roman-numbers-converter-sec-grp --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-names umut-roman-numbers-converter-sec-grp --protocol tcp --port 22 --cidr 0.0.0.0/0
```

3. After creating security group, we will create an instance using latest ami version. To do this, we need to find description with AWS system manager (ssm) command:
- The command below will retrieve the description of latest AMI image(Linux 2):
```bash
aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --region us-east-1 
```
- This command is to run querry to get latest AMI ID (Linux 2):
```bash
aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --query 'Parameters[0].[Value]' --output text
```
- Then, we will need to assign this AMI id to the LATEST_AMI environmental variable:
```bash
LATEST_AMI=$(aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --query 'Parameters[0].[Value]' --output text)
```
- Now, we can create a userdata.sh file under /home/ec2-user to run the commands for installing python, flask, git and get files from github. And it will run our python app in our new instance:
```bash
#! /bin/bash
          yum update -y
          yum install python3 -y
          pip3 install flask
          yum install git -y
          cd /home/ec2-user
          FOLDER="https://raw.githubusercontent.com/umut-burdur/my-projects/refs/heads/main/aws/projects/001-roman-numerals-converter/"
          wget ${FOLDER}/roman-numerals-converter-app.py
          mkdir templates && cd templates
          wget ${FOLDER}/templates/index.html
          wget ${FOLDER}/templates/result.html
          cd ..
          python3 roman-numerals-converter-app.py
```
Then we will create new instance with the line below:
```bash
aws ec2 run-instances --image-id $LATEST_AMI --count 1 --instance-type t2.micro --key-name first-key --security-groups umut-roman-numbers-converter-sec-grp --tag-specifications 'ResourceType=string,Tags=[{Key=Name,Value=umut_roman_number_converter}]' --user-data file:///home/ec2-user/userdata.sh
```
- To see the each created instance(s) and information, we will use describe instance CLI command:
```bash
aws ec2 describe-instances --filters "Name=tag:Name,Values=umut_roman_number_converter"
```
- To see Public IP and instance_id of instances, run the query below:
```bash
aws ec2 describe-instances --filters "Name=tag:Name,Values=umut_roman_number_converter" --query 'Reservations[].Instances[].PublicIpAddress[]'
aws ec2 describe-instances --filters "Name=tag:Name,Values=umut_roman_number_converter" --query 'Reservations[].Instances[].InstanceId[]'
```
- To delete instances:
```bash
aws ec2 terminate-instances --instance-ids <instance-id which we get above>
```
- To delete security groups:
```bash
aws ec2 delete-security-group --group-name umut-roman-numbers-converter-sec-grp
```
