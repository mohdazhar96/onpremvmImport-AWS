# onpremvmImport-AWS
importing on prem VM into the aws ec2.

Prerequisites for importing a VM into Amazon EC2

    Create an Amazon Simple Storage Service (Amazon S3) bucket for storing the exported images or choose an existing bucket.
    Create an IAM role named vmimport
    
Upload the image to Amazon S3
    
     Upload your VM image file to your S3 bucket using the upload tool of your choice
    
The following examples use the AWS CLI command import-image to create import tasks.

Example 1: Import an image with a single disk

Use the following command to import an image with a single disk.

     aws ec2 import-image --description "My server VM" --disk-containers "file://C:\import\containers.json"

The following is an example containers.json file that specifies the image using an S3 bucket.

     The following examples use the AWS CLI command import-image to create import tasks.

Example 1: Import an image with a single disk

Use the following command to import an image with a single disk.

    aws ec2 import-image --description "My server VM" --disk-containers "file://C:\import\containers.json"
now, You want to import a virtual machine (VM) from your on-premises data center to AWS EC2.
But to do that, you need to give AWS VM Import/Export service permission to:

    Access your S3 buckets (where your VM files are stored),

    And do things like register images and snapshots in EC2.

So you need to create a helper (IAM Role) for the VM Import service to use.


reference link: https://docs.aws.amazon.com/vm-import/latest/userguide/required-permissions.html#vmimport-role


To create the service role

    Create a file named trust-policy.json on your computer. Add the following policy to the file:

{
   "Version": "2012-10-17",
   "Statement": [
      {
         "Effect": "Allow",
         "Principal": { "Service": "vmie.amazonaws.com" },
         "Action": "sts:AssumeRole",
         "Condition": {
            "StringEquals":{
               "sts:Externalid": "vmimport"
            }
         }
      }
   ]
}


Use the create-role command to create a role named vmimport and grant VM Import/Export access to it. Ensure that you specify the full path to the location of the trust-policy.json file that you created in the previous step, and that you include the file:// prefix as shown the following example:

aws iam create-role --role-name vmimport --assume-role-policy-document "file://C:\import\trust-policy.json"

Create a file named role-policy.json with the following policy, where amzn-s3-demo-import-bucket is the bucket for imported disk images and amzn-s3-demo-export-bucket is the bucket for exported disk images:

{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Effect": "Allow",
         "Action": [
            "s3:GetBucketLocation",
            "s3:GetObject",
            "s3:ListBucket" 
         ],
         "Resource": [
            "arn:aws:s3:::amzn-s3-demo-import-bucket",
            "arn:aws:s3:::amzn-s3-demo-import-bucket/*"
         ]
      },
      {
         "Effect": "Allow",
         "Action": [
            "s3:GetBucketLocation",
            "s3:GetObject",
            "s3:ListBucket",
            "s3:PutObject",
            "s3:GetBucketAcl"
         ],
         "Resource": [
            "arn:aws:s3:::amzn-s3-demo-export-bucket",
            "arn:aws:s3:::amzn-s3-demo-export-bucket/*"
         ]
      },
      {
         "Effect": "Allow",
         "Action": [
            "ec2:ModifySnapshotAttribute",
            "ec2:CopySnapshot",
            "ec2:RegisterImage",
            "ec2:Describe*"
         ],
         "Resource": "*"
      }
   ]
}
# https://www.youtube.com/watch?v=tdck3TReQcM
*/
