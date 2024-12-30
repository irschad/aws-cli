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
- AWS CLI installed on the system. ([Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html))
- Linux environment.

---

## Step 1: Configure AWS CLI
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
--cidr 227.231.27.134/32
```
```json
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0a2b4ba619a4071ea",
            "GroupId": "sg-0f366dec7784c7264",
            "GroupOwnerId": "922854651834",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIpv4": "227.231.27.134/32"
        }
    ]
}
```
### Describe Security Group:
```bash
aws ec2 describe-security-groups --group-ids sg-0f366dec7784c7264
```
```json
{
    "SecurityGroups": [
        {
            "Description": "My SG",
            "GroupName": "my-sg",
            "IpPermissions": [
                {
                    "FromPort": 22,
                    "IpProtocol": "tcp",
                    "IpRanges": [
                        {
                            "CidrIp": "227.231.27.134/32"
                        }
                    ],
                    "Ipv6Ranges": [],
                    "PrefixListIds": [],
                    "ToPort": 22,
                    "UserIdGroupPairs": []
                }
            ],
            "OwnerId": "922854651834",
            "GroupId": "sg-0f366dec7784c7264",
            "IpPermissionsEgress": [
                {
                    "IpProtocol": "-1",
                    "IpRanges": [
                        {
                            "CidrIp": "0.0.0.0/0"
                        }
                    ],
                    "Ipv6Ranges": [],
                    "PrefixListIds": [],
                    "UserIdGroupPairs": []
                }
            ],
            "VpcId": "vpc-02b046adc6650bdb6"
        }
    ]
}
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
#### Restrict permissions:
```bash
chmod 400 MyKpCli.pem
```

---

## Step 4: Launch an EC2 Instance

Check AMI ID to be used at AWS console from Launch Instances -> Quick Start AMIs, and copy the AMI ID and use in below command.
![Quick Start AMIs](https://github.com/user-attachments/assets/8d3d4441-e21f-44a2-b619-94f1113d1d5b)


```bash
aws ec2 run-instances \
--image-id ami-01816d07b1128cd2d \
--count 1 \
--instance-type t2.micro \
--key-name MyKpCli \
--security-group-ids sg-0f366dec7784c7264 \
--subnet-id subnet-03d3ed0a20397d803
```
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
## Step 5: SSH into the new EC2 instance
```bash
$ ssh -i MyKpCli.pem ec2-user@44.222.166.107
The authenticity of host '44.222.166.107 (44.222.166.107)' can't be established.
ED25519 key fingerprint is SHA256:0pBybF7Vmzpw+xKMkWANPKVzNiHwRRlL4glFVBAmlzao.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '44.222.166.107' (ED25519) to the list of known hosts.
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
$ exit
```
---
## Step 6: Filter and Query 
```bash
aws ec2 describe-instances --filters "Name=instance-type,Values=t2.micro" --query "Reservations[].Instances[].InstanceId"
```
```json
[
    "i-0e4594f32c6dbe39a",
    "i-00c8bbe17ceaa4a4f"
]
```
---

## Step 7: Manage IAM Resources

### Create a Group:
```bash
aws iam create-group --group-name MyGroupCli
```
```json
{
    "Group": {
        "Path": "/",
        "GroupName": "MyGroupCli",
        "GroupId": "AGPA5NXTMO65DSTTOPCZI",
        "Arn": "arn:aws:iam::922854651834:group/MyGroupCli",
        "CreateDate": "2024-10-29T11:55:37+00:00"
    }
}
```

### Create a User:
```bash
aws iam create-user --user-name MyUserCli
```
```json
{
    "User": {
        "Path": "/",
        "UserName": "MyUserCli",
        "UserId": "AIDA5NXTMO65HDDV67IYJ",
        "Arn": "arn:aws:iam::922854651834:user/MyUserCli",
        "CreateDate": "2024-10-29T11:56:14+00:00"
    }
}
```

### Add User to Group:
```bash
aws iam add-user-to-group --user-name MyUserCli --group-name MyGroupCli
```
```bash
aws iam get-group --group-name MyGroupCli
```
```json
{
    "Users": [
        {
            "Path": "/",
            "UserName": "MyUserCli",
            "UserId": "AIDA5NXTMO65HDDV67IYJ",
            "Arn": "arn:aws:iam::922854651834:user/MyUserCli",
            "CreateDate": "2024-10-29T11:56:14+00:00"
        }
    ],
    "Group": {
        "Path": "/",
        "GroupName": "MyGroupCli",
        "GroupId": "AGPA5NXTMO65DSTTOPCZI",
        "Arn": "arn:aws:iam::922854651834:group/MyGroupCli",
        "CreateDate": "2024-10-29T11:55:37+00:00"
    }
}
```

### Attach Policy to Group:
```bash
- Get ARN of AmazonEC2FullAccess policy
aws iam list-policies --query 'Policies[?PolicyName==`AmazonEC2FullAccess`].Arn' --output text
arn:aws:iam::aws:policy/AmazonEC2FullAccess

- Attach the group policy
aws iam attach-group-policy --group-name MyGroupCli --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess
```

- List attached group policies
```bash
aws iam list-attached-group-policies --group-name MyGroupCli
{
    "AttachedPolicies": [
        {
            "PolicyName": "AmazonEC2FullAccess",
            "PolicyArn": "arn:aws:iam::aws:policy/AmazonEC2FullAccess"
        }
    ]
}
```

---

### Create Credentials for new User
- Create Login profile
```bash
aws iam create-login-profile --user-name MyUserCli --password MyPassword! --password-reset-required
```
```json
{
    "LoginProfile": {
        "UserName": "MyUserCli",
        "CreateDate": "2024-10-29T12:14:13+00:00",
        "PasswordResetRequired": true
    }
}
```
- Get account ID:
```bash
aws iam get-user --user-name MyUserCli
```
```json
{
    "User": {
        "Path": "/",
        "UserName": "MyUserCli",
        "UserId": "AIDA5NXTMO65HDDV67IYJ",
        "Arn": "arn:aws:iam::922854651834:user/MyUserCli",
        "CreateDate": "2024-10-29T11:56:14+00:00"
    }
}
```

### Create Policy and assign to group
- Write policy for password change.
  Refer ([User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_passwords_enable-user-change.html))
```bash
vi changePwdpolicy.json
```
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iam:GetAccountPasswordPolicy",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "iam:ChangePassword",
      "Resource": "arn:aws:iam::*:user/${aws:username}"
    }
  ]
}
```
- Create policy

```bash
aws iam create-policy --policy-name changePwd --policy-document file://changePwdPolicy.json
```
```json
{
    "Policy": {
        "PolicyName": "changePwd",
        "PolicyId": "ANPA5NXTMO65BHTV3XUB4",
        "Arn": "arn:aws:iam::922854651834:policy/changePwd",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2024-10-29T12:23:37+00:00",
        "UpdateDate": "2024-10-29T12:23:37+00:00"
    }
}
```
- Attach group policy
```bash
aws iam attach-group-policy --group-name MyGroupCli --policy-arn arn:aws:iam::922854651834:policy/changePwd
```
- Test it by logging it to AWS console with this user. Change the password and login with new password.
![change password](https://github.com/user-attachments/assets/f96854cd-c83e-48d9-a12d-4c4506a627a8)

---

### Create Access Keys for a new User
```bash
aws iam create-access-key --user-name MyUserCli
```
```json
{
    "AccessKey": {
        "UserName": "MyUserCli",
        "AccessKeyId": "AKIxxxxxxxxxxxxx",
        "Status": "Active",
        "SecretAccessKey": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        "CreateDate": "2024-10-29T14:25:01+00:00"
    }
}
```

### Switch AWS Users for AWS CLI commands
--------------------------------------
- Switch to newly created AWS user
export AWS_ACCESS_KEY_ID=AKIxxxxxxxxxxxxx
export AWS_SECRET_ACCESS_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

export AWS_DEFAULT_REGION=eu-west-2

- Test run command to create another user as a new user
```bash 
aws iam create-user --user-name test
```
Notice that this results in access denied error while attempting to create another user as the new user created doesn't have access for same.

## Step 8: Cleanup - Delete Resources



### Detach group policy:
```bash
aws iam detach-group-policy --group-name MyGroupCli --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess
```

### Delete policy:
```bash
aws iam delete-policy --policy-arn arn:aws:iam::922854651834:policy/changePwd
```

### Delete login profile:
```bash
aws iam delete-login-profile --user-name MyUserCli
```

### Remove user from group
```bash
aws iam remove-user-from-group --user-name MyUserCli --group-name MyGroupCli
```

### Delete access key
```bash
aws iam delete-access-key --user-name MyUserCli --access-key-id AKIxxxxxxxxxxxxx
```

### Delete user
```bash
aws iam delete-user --user-name MyUserCli
```

### Delete group
```bash
aws iam delete-group --group-name MyGroupCl
```

### Terminate EC2 Instances:
```bash
aws ec2 terminate-instances --instance-ids i-00c8bbe17ceaa4a4f
```
```json
{
    "TerminatingInstances": [
        {
            "CurrentState": {
                "Code": 32,
                "Name": "shutting-down"
            },
            "InstanceId": "i-00c8bbe17ceaa4a4f",
            "PreviousState": {
                "Code": 16,
                "Name": "running"
            }
        }
    ]
}
```
Verify from AWS console that instance is terminated.

![Instances | EC2 | us-east-1](https://github.com/user-attachments/assets/86883311-d6aa-4a6c-8def-d75a643c9cd2)


### Delete Security Group:
```bash
aws ec2 delete-security-group --group-id sg-0f366dec7784c7264
```

### Delete Key Pair:
```bash
aws ec2 delete-key-pair --key-name MyKpCli
```
```json
{
    "Return": true,
    "KeyPairId": "key-09857c346ba9fcd51"
}
```


---

## Conclusion
This project demonstrates how to effectively interact with AWS resources using the AWS CLI, ensuring resource creation, management, and cleanup are streamlined and efficient.

