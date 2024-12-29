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
```json
--
{
    "Instances": [
        {
            "InstanceId": "i-00c8bbe17ceaa4a4f",
            "State": { "Name": "pending" },
            "PrivateIpAddress": "10.0.42.253"
        }
    ]
}
--
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
        "CreateDate": "2024-12-29T11:56:14+00:00"
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
- Write policy for password change
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
---

### Create Access Keys for a new User
```bash
aws iam create-access-key --user-name MyUserCli
```

## Step 8: Cleanup - Delete Resources
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

