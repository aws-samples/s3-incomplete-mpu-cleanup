# Amazon S3 Incomplete Multipart Upload cleanup

## Amazon Multipart Upload

When uploading large objects to an Amazon S3 bucket, it is recommended to use Amazon S3 multipart upload to improve performance. Many mechanisms like AWS CLI, SDKs, and the AWS Console use multipart upload automatically for large objects. Multipart upload splits the object into smaller chunks and uploads each chunk in parallel. The chunks are reassembled into the full object when they reach the Amazon S3 bucket. However, network issues may cause some chunks to fail during upload. These failed chunks still reside in the S3 bucket, incurring storage costs even though the data is unusable. This can lead to unnecessary S3 costs over time, especially for very large buckets. The incomplete uploads are not visible in the AWS Console except through low-level S3 API commands, which can be challenging.

To delete incomplete multipart uploads, S3 lifecycle policies are recommended. However, organizations with many S3 buckets may struggle to implement policies consistently. This solution checks for buckets without lifecycle policies deleting incomplete uploads, marking them non-compliant. A Systems Manager document created by the solution then remediates non-compliant buckets by applying a lifecycle policy to remove incomplete multipart uploads after 7 days. The solution also detects and remediates new S3 buckets automatically.


## Solution Overview

The solution involves creating a custom rule in AWS Config that regularly checks all S3 buckets in the account using a custom Guard policy. The rule verifies if buckets have a lifecycle policy to abort incomplete multipart uploads. Buckets without this policy are marked as non-compliant in AWS Config. The solution also uses a custom Python and Boto3 SSM document for remediation. The SSM document takes non-compliant buckets from AWS Config and applies a lifecycle policy to abort incomplete multipart uploads after 7 days, while retaining existing lifecycle policies transitioning or expiring objects. The solution also avoids listing buckets that already have a policy to abort incomplete multipart uploads as non-compliant.

![image1](https://github.com/aws-samples/s3-incomplete-mpu-cleanup/blob/main/image1.png)

## Prerequisite 

AWS Config needs to be setup before deploying the CloudFormation template for this solution. For more information, see Setting up [AWS Config](https://docs.aws.amazon.com/config/latest/developerguide/1-click-setup.html).

## Implementation

We will utilize AWS CloudFormation to provision the resources needed for this solution. Specifically, CloudFormation will create an AWS Config custom rule, a Systems Manager remediation document, and an IAM role granting Systems Manager permission to implement an S3 lifecycle policy. The template will also connect the SSM document as a remediation action for the AWS Config rule that was created.


The CloudFormation YAML template in this repository can be uploaded to the AWS CloudFormation console to deploy the solution stack, if you do not want to use the CLI instructions below.


### A) Clone the repository
```
git clone https://github.com/aws-samples/s3-incomplete-mpu-cleanup.git
```
### B) Switch to the repository's directory 
```
cd s3-incomplete-mpu-cleanup
```

### C) Create a satck deploy solution
```
aws cloudformation create-stack \
	--stack-name S3-MPU-Delete \
	--template-body file://s3mpu-delete.yaml --capabilities CAPABILITY_IAM
```


## Further configuration
The config rule created by the CloudFormation template is not working as expected. Although the template set the AWS S3 Bucket resource type for the rule, testing shows the rule does not work unless the resource type is manually reselected and saved in the console. This indicates an issue with how CloudFormation and Config are interacting that needs further investigation. 

## Steps:

1. Click on the Edit Button, scroll to Evaluation mode section.
2. Make sure Resource radio button is checked
3. In the Resource category, select AWS Resources
4. In Resource Type, search for s3 and select AWS S3 Bucket.
5. Click the Save button.

You can either wait for the config rule evaluation run or you can manually trigger the evaluation by click on Action and select Re-evaluate.


