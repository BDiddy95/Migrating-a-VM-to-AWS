# Migrating-a-VM-to-AWS
This project portrays the procedure necessary to move a Linux VMI running on a windows host to AWS.

Steps to migrate VM to AWS

Pre-reqs

A Virtual Machine

AWS CLI 2.0 installed on your system

Step 1

Export the Virtual Machine Image to a file on your computer. Ensure that the format is .vmdk or .ova

The bucket must be in the region where you want to import your VMs.

Create an S3 bucket and upload the image file to this bucket.

--
aws s3 mb s3://[Insert-Bucket-Name]

aws s3api put-object --bucket [Bucket you created] --key [Vm Image file]  --body [Insert File Path (If using windows)] ** 

--

Create the IAM Role for VM Import

Create the IAM policy: role-policy.json

Attach policy to IAM Role: vmimport




aws iam create-role --role-name vmimport --assume-role-policy-document [Insert Trust policy.Json file path]

aws iam put-role-policy --role-name vmimport --policy-name vmimport --policy-document "[Insert role policy.json file path]"

aws ec2 import-image --description "My server disks" --disk-containers "[filepath/containers.json]"

aws ec2 describe-import-image-tasks --import-task-ids import-ami-07d69203e50f95609

				Trust-Policy.json
===================================================================================
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "vmie.amazonaws.com"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:Externalid": "vmimport"
        }
      }
    }
  ]
}
===================================================================================


				Role-Policy.json
===================================================================================
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetBucketLocation",
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::[Bucket-Name]",
        "arn:aws:s3:::[Bucket-Name]/*"
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
===================================================================================

				Containers.json

===================================================================================
[
{
"Description": "[Insert whatever your heart desires] ",
"Format": "ova",
"UserBucket": {
"S3Bucket": "[Insert Bucket Name]",
"S3Key": "[Insert VMIMage file name]"
}
}
]
========================================================================================
