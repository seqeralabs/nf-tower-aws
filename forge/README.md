# Tower Forge for AWS Batch

Tower Forge automates the configuration of [AWS Batch](https://aws.amazon.com/batch/) compute environments and queues
required for the deployment of Nextflow pipelines. 

To enable this feature Tower requires the permissions listed in [this policy](forge-policy.json) file. 

Attach the policy to the AWS user account associated to your Tower configuration as described below: 

1) Open the AWS [IAM console](https://console.aws.amazon.com/iam/home)
2) Select *Users* on the left menu 
3) Choose (or create) the user associated in your Tower configuration 
4) Click on *Add inline policy*
5) Choose *JSON* and copy the content of the policy linked above. 
6) Click on the button *Review policy* and confirm the operation clicking *Create policy* 

Note: This policy also includes the mininal permissions required to allow the user to submit
Batch jobs, gather containers execution metadata, read CloudWatch logs and access the S3 bucket in your AWS 
account in read-only mode. 

You may need to further customised the IAM permissions to access private ECR registries, 
write to S3 buckets or access other AWS resources. 

### Pipeline Secrets

If you are planning to use the Pipeline Secrets feature provided by Tower, the following
IAM permissions should be provided:

Add the custom policy at [this link](../launch/secrets-policy-account.json) to the IAM user or role granting
   access to your AWS account to Tower (the one specified in the Tower credentials).

See [Tower Launch](../launch/README.md) for more details.
