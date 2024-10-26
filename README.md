# DeepakGym S3 Static Website

## Description
This project involves setting up an Amazon EC2 instance and configuring an S3 bucket to host a static website using a CSS template. The S3 bucket is configured for public access and static website hosting.

## Prerequisites
- AWS CLI installed and configured
- An AWS account
- SSH access to EC2 instances

## Installation Steps

### Step 1: Create an IAM Role
1. Create a trust policy file named `trust-policy.json`:
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
    ```

2. Create the IAM role:
    ```bash
    aws iam create-role --role-name EC2S3FullAccessRole --assume-role-policy-document file://trust-policy.json
    ```

3. Create an S3 full access policy file named `s3-full-access-policy.json`:
    ```json
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
    ```

4. Attach the policy to the role:
    ```bash
    aws iam put-role-policy --role-name EC2S3FullAccessRole --policy-name S3FullAccessPolicy --policy-document file://s3-full-access-policy.json
    ```

### Step 2: Create an Instance Profile
1. Create an instance profile:
    ```bash
    aws iam create-instance-profile --instance-profile-name EC2S3FullAccessProfile
    ```

2. Add the role to the instance profile:
    ```bash
    aws iam add-role-to-instance-profile --instance-profile-name EC2S3FullAccessProfile --role-name EC2S3FullAccessRole
    ```

### Step 3: Launch an EC2 Instance with the IAM Role Attached
1. Launch the EC2 instance:
    ```bash
    aws ec2 run-instances --image-id ami-07c5ecd8498c59db5 --instance-type t2.micro --key-name s3keypair --iam-instance-profile Name=EC2S3FullAccessProfile
    ```

2. Connect to the instance using SSH:
    ```bash
    ssh -i s3keypair.pem ec2-user@your-ec2-instance-public-ip
    ```

### Step 4: Create an S3 Bucket
1. Create an S3 bucket:
    ```bash
    aws s3 mb s3://deepakgym.com
    ```

### Step 5: Install Unzip and Download the Web Template
1. Install unzip:
    ```bash
    sudo yum install -y unzip
    ```

2. Download the template:
    ```bash
    curl -O https://www.free-css.com/assets/files/free-css-templates/download/page296/neogym.zip
    ```

3. Unzip the template:
    ```bash
    unzip neogym.zip -d neogym-html
    ```

### Step 6: Copy Files to S3 Bucket
1. Copy files from the local directory to the S3 bucket:
    ```bash
    aws s3 cp neogym-html/ s3://deepakgym.com/ --recursive
    ```

### Step 7: Remove Public Access Block on the Bucket
1. Remove public access block:
    ```bash
    aws s3api delete-public-access-block --bucket deepakgym.com
    ```

### Step 8: Enable Static Website Hosting on S3
1. Enable static website hosting:
    ```bash
    aws s3 website s3://deepakgym.com --index-document index.html --error-document error.html
    ```

### Step 9: Create a Bucket Policy for Public Access
1. Create a bucket policy JSON file:
    ```bash
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
    EOL
    ```

2. Apply the bucket policy:
    ```bash
    aws s3api put-bucket-policy --bucket deepakgym.com --policy file://policy.json
    ```

For accessing the Application from web 
    
    ```bash
    http://deepakgym.com.s3.<region>.amazonaws.com/<object-key>
    ```
    
## Conclusion
Now the static website should now be accessible through the S3 bucket. Ensure that the bucket policy and the public access settings are correctly configured to allow public access.

