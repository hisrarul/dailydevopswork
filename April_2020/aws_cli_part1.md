Q.1 Create IAM user using aws cli and name of the user is "devops".
```
user='devops'
aws iam create-user --user-name $user
```

Q.2 Create IAM group using aws cli and name of the group is "devops".
```
group='devops'
aws iam create-group --group-name $group
```


Q.3 Create IAM Policy using aws cli for the group "devops" so that each user in the group can list all instances and start them.
```
cat policy.txt
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "ec2:StartInstances",
                "ec2:CreateTags"
            ],
            "Resource": "arn:aws:ec2:*:*:instance/*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeInstances",
                "ec2:DescribeTags"
            ],
            "Resource": "*"
        }
    ]
}
```
> Create an IAM policy \
aws iam create-policy --policy-name devops-policy --policy-document file://policy.txt

> Attach an IAM policy with the group devops \
aws iam attach-group-policy --policy-arn arn:aws:iam::462453535472:policy/devops-policy --group-name devops


Q.4 Create IAM role using the aws cli and attach "ReadOnlyAccess" policy with the role.
Name=devops-iam-profile
```
aws iam create-role --role-name devops-iam-profile --assume-role-policy-document file://policy.txt

aws iam attach-role-policy --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess --role-name devops-iam-profile

aws iam create-instance-profile --instance-profile-name devops-iam-profile

aws iam add-role-to-instance-profile --instance-profile-name devops-iam-profile --role-name devops-iam-profile
```
#### Note:
> (Answer of today's question): \
If you use the AWS Management Console to create a role for Amazon EC2, the console automatically creates an instance profile and gives it the same name as the role. When you then use the Amazon EC2 console to launch an instance with an IAM role, you can select a role to associate with the instance. In the console, the list that's displayed is actually a list of instance profile names. The console does not create an instance profile for a role that is not associated with Amazon EC2.


## Using for loop

>vi role_name.txt \
Enter the name of the role e.g. devops-iam-profile

```
cat policy.json 
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}

for name in $(cat role_name.txt)
do 
aws iam create-role --role-name $name --assume-role-policy-document file://policy.json
aws iam attach-role-policy --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess --role-name $name
aws iam create-instance-profile --instance-profile-name $name
aws iam add-role-to-instance-profile --instance-profile-name $name --role-name $name
echo "IAM role named $name has been created.."
done
```



Q.5 Create VPC with a cidr range of 40.0.0.0/16 using aws cli and add a tag as defined.
Name=Asgad

```
aws ec2 create-vpc \
    --instance-tenancy default \
    --cidr-block 40.0.0.0/16
```
aws ec2 create-tags --resources vpc-07e62773bea22eafe --tags Key=Name,Value=Asgad


Q.6 Create Internet Gateway and attach it with the vpc named "Asgad"
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --internet-gateway-id igw-052d5791ae8db19c1 --vpc-id vpc-07e62773bea22eafe

Q.7 Create two subnets with following details
subnet1 name = Public-Asgad
cidr range = 40.0.1.0/24

subnet2 name = Private-Asgad
cidr range = 40.0.2.0/24


aws ec2 create-subnet --vpc-id $vpcId --cidr-block 40.0.1.0/24
```
for i in $(seq 0 5)
do 
value=$(aws ec2 describe-vpcs --query Vpcs[$i].Tags[0].Value)
echo $value
if [ $value == \"Asgad\" ] 
then vpcId=$(aws ec2 describe-vpcs --query Vpcs[$i].VpcId | cut -d'"' -f2) 
echo $vpcId 
aws ec2 create-subnet --vpc-id $vpcId --cidr-block 40.0.1.0/24 
fi
done
```

Q.8 Create the ec2 instance with IAM profile using aws cli.
```
aws ec2 run-instances \
    --image-id ami-0470e33cd681b2476 \
    --count 1 \
    --instance-type t2.micro \
    --key-name pc-israr-k8s \
    --iam-instance-profile Name=devops-iam-profile \
    --subnet-id subnet-068631b9d21d9d591
```
#### Prepared by Israrul Haque
Follow me on Twitter: @hisrarul \
Reserved Copyrights
