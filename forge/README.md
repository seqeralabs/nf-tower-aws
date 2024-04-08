# Batch Forge for AWS Batch

Batch Forge automates the configuration of [AWS Batch](https://aws.amazon.com/batch/) compute environments and queues
required for the deployment of Nextflow pipelines. 

To enable this feature, Seqera Platform requires the permissions listed in [this policy](forge-policy.json) file. 

Attach the policy to the AWS user account associated to your Seqera configuration as described below: 

1) Open the AWS [IAM console](https://console.aws.amazon.com/iam/home) and select **Users**.
2) Select or create the user associated with your Seqera configuration.
3) Select **Add inline policy**.
4) Select **JSON** and copy the content of the policy linked above. 
6) Select **Review policy** and then **Create policy**.

> **Note** 
> This policy also includes the mininal permissions required to allow the user to submit
> Batch jobs, gather container execution metadata, read CloudWatch logs, and access the S3 bucket in your AWS 
> account in read-only mode. 

> **Important**
> You may need to customize the IAM permissions to access private ECR registries, 
> write to S3 buckets, or access other AWS resources. 

### Pipeline secrets

To use pipeline secrets in Seqera Platform, the following
IAM permissions must be provided:

Add [this custom policy](../launch/secrets-policy-account.json) to the IAM user or role used by Seqera
   to access your AWS account (specified in the Seqera credentials).

See [Seqera Launch](../launch/README.md) for more details.
