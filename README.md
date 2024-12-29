# Interacting with AWS CLI

## Project Overview
This project demonstrates how to interact with AWS resources using the AWS Command Line Interface (CLI) on a Linux system. The key objectives include:

- Installing and configuring the AWS CLI.
- Creating an EC2 instance with appropriate configurations such as a security group and key pair.
- Managing IAM resources like Users, Groups, and Policies.
- Filtering and querying AWS resources.
- Deleting resources as a cleanup step.

---

## Prerequisites
- AWS account credentials.
- AWS CLI installed on your system. ([Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html))
- Linux environment.

---

## Step 1: Install and Configure AWS CLI
### Command:
```bash
aws configure
```
### Output:
```bash
cat ~/.aws/config
[default]
region = us-east-1
output = json

cat ~/.aws/credentials
aws_access_key_id=********************
aws_secret_access_key=************************
```

---

## Step 2: Create a Security Group
### Describe VPCs:
```bash
aws ec2 describe-vpcs --query "Vpcs[].VpcId"
```
#### Output:
```json
[
    "vpc-094e263a5e8aa7dca",
    "vpc-02b046adc6650bdb6"
]
```

### Create Security Group:
```bash
aws ec2 create-security-group --group-name my-sg --description "My SG" --vpc-id vpc-02b046adc6650bdb6
```
#### Output:
```json
{
    "GroupId": "sg-0f366dec7784c7264"
}
```

### Authorize Inbound SSH Access:
```bash
aws ec2 authorize-security-group-ingress \
--group-id sg-0f366dec7784c7264 \
--protocol tcp \
--port 22 \
--cidr YOUR_IP_ADDRESS/32
```

---

## Step 3: Create a Key Pair
### Command:
```bash
aws ec2 create-key-pair \
--key-name MyKpCli \
--query 'KeyMaterial' \
--output text > MyKpCli.pem
```
#### Adjust Permissions:
```bash
chmod 400 MyKpCli.pem
```

---

## Step 4: Launch an EC2 Instance
### Command:
```bash
aws ec2 run-instances \
--image-id ami-01816d07b1128cd2d \
--count 1 \
--instance-type t2.micro \
--key-name MyKpCli \
--security-group-ids sg-0f366dec7784c7264 \
--subnet-id subnet-03d3ed0a20397d803
```
#### Output:
```json
{
    "Instances": [
        {
            "InstanceId": "i-00c8bbe17ceaa4a4f",
            "State": { "Name": "pending" },
            "PrivateIpAddress": "10.0.42.253"
        }
    ]
}
```

---

## Step 5: Manage IAM Resources
### Create a Group:
```bash
aws iam create-group --group-name MyGroupCli
```
### Create a User:
```bash
aws iam create-user --user-name MyUserCli
```
### Add User to Group:
```bash
aws iam add-user-to-group --user-name MyUserCli --group-name MyGroupCli
```
### Attach Policy to Group:
```bash
aws iam attach-group-policy --group-name MyGroupCli --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess
```

---

## Step 6: Query AWS Resources
### Describe Instances:
```bash
aws ec2 describe-instances --query "Reservations[].Instances[].InstanceId"
```
#### Output:
```json
[
    "i-00c8bbe17ceaa4a4f"
]
```

---

## Step 7: Cleanup - Delete Resources
### Terminate EC2 Instances:
```bash
aws ec2 terminate-instances --instance-ids i-00c8bbe17ceaa4a4f
```

### Delete Security Group:
```bash
aws ec2 delete-security-group --group-id sg-0f366dec7784c7264
```

### Delete Key Pair:
```bash
aws ec2 delete-key-pair --key-name MyKpCli
```

### Detach and Delete IAM Group:
```bash
aws iam detach-group-policy --group-name MyGroupCli --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess
aws iam delete-group --group-name MyGroupCli
```

### Delete IAM User:
```bash
aws iam delete-user --user-name MyUserCli
```

---

## Conclusion
This project demonstrates how to effectively interact with AWS resources using the AWS CLI, ensuring resource creation, management, and cleanup are streamlined and efficient.

