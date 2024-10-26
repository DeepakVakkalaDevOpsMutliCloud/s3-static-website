# s3-static-website
deepak-gym-website

## Setup Instructions

### Step 1: Create an IAM Role for EC2 with S3 Full Access

#### Create a Trust Policy
Create a trust policy file named `trust-policy.json` that allows EC2 to assume this role. Save the following JSON content to that file:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "ec2.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}


Create the IAM Role
Run this command to create the IAM role:

aws iam create-role --role-name EC2S3FullAccessRole --assume-role-policy-document file://trust-policy.json

Create the S3 Full Access Policy
Create a policy file named s3-full-access-policy.json with the following content:

json

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:*",
                "s3-object-lambda:*"
            ],
            "Resource": "*"
        }
    ]
}

Attach the Policy to the Role
You can attach the policy you just created with the following command:


aws iam put-role-policy --role-name EC2S3FullAccessRole --policy-name S3FullAccessPolicy --policy-document file://s3-full-access-policy.json


Step 2: Create an Instance Profile and Attach the Role
Create an Instance Profile
Create an instance profile for the EC2 instance:

aws iam create-instance-profile --instance-profile-name EC2S3FullAccessProfile
Add the Role to the Instance Profile
Add the role to the instance profile:


aws iam add-role-to-instance-profile --instance-profile-name EC2S3FullAccessProfile --role-name EC2S3FullAccessRole
Step 3: Launch an EC2 Instance with the IAM Role Attached
When creating the EC2 instance, specify the instance profile so that it has access to S3. Here’s the command to launch the instance:

aws ec2 run-instances --image-id ami-07c5ecd8498c59db5 --instance-type t2.micro --key-name s3keypair --ia

Step 1: Create an EC2 Instance
Run this command to create an EC2 instance with Amazon Linux:


aws ec2 run-instances --image-id ami-07c5ecd8498c59db5 --instance-type t2.micro --key-name s3keypair --iam-instance-profile Name=EC2S3FullAccessProfile
Once created, connect to the instance using SSH:


ssh -i s3keypair.pem ec2-user@your-ec2-instance-public-ip
Step 2: Create an S3 Bucket
Create an S3 bucket named deepakgym.com:


aws s3 mb s3://deepakgym.com
Step 3: Install Unzip and Download the Web Template
Install unzip:


sudo yum install -y unzip
Download the template from the web:


curl -O https://www.free-css.com/assets/files/free-css-templates/download/page296/neogym.zip
Unzip the template:


unzip neogym.zip -d neogym-html
Step 4: Copy Files to S3 Bucket
Copy all files from the neogym-html directory to the S3 bucket:

aws s3 cp neogym-html/ s3://deepakgym.com/ --recursive
Step 5: Remove Public Access Block on the Bucket
Remove the S3 bucket’s public access block:

aws s3api delete-public-access-block --bucket deepakgym.com
Step 6: Enable Static Website Hosting on S3
Enable static website hosting for the S3 bucket:


aws s3 website s3://deepakgym.com --index-document index.html --error-document error.html
Step 7: Create a Bucket Policy for Public Access
Create a bucket policy JSON file:


cat > policy.json << EOL
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::deepakgym.com/*"
        }
    ]
}

Apply the bucket policy:


aws s3api put-bucket-policy --bucket deepakgym.com --policy file://policy.json



