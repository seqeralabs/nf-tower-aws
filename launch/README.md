# Launch for AWS Batch

Seqera Platform enables the seamless deployment of Nextflow data pipelines with the [AWS Batch](https://aws.amazon.com/batch/) computing service. 

1. Create a AWS user account (or use an existing one) granting at least the following IAM permission: 

  - `AmazonS3ReadOnlyAccess` 
  - `AmazonEC2ContainerRegistryReadOnly`
  - `CloudWatchLogsReadOnlyAccess` 
  - The following [custom policy](launch-policy.json) to grant the ability to submit and control Batch jobs.
  - Grant write access to any S3 bucket used as a pipeline work directory with the following [policy template](s3-bucket-write.json). 

1. Create the AWS Batch queue(s) required to deploy the Nextflow execution. 

Seqera can automate this configuration step and grant the required permissions with [Batch Forge](../forge/README.md).

### Seqera role trust policy (optional)

You can optionally create a Seqera role trust policy to allow EC2 instances or EKS clusters (depending on your Seqera deployment) to assume the Seqera IAM role.

1. Download the [Seqera role trust policy](../launch/seqera-role-trust-policy.json).
1. Replace `YOUR-AWS-ACCOUNT` with your AWS account ID. 
1. Replace `USER-OR-ROLE/USER-OR-ROLE-ID` with the users and or roles that must be able to assume the Seqera IAM role. 

### Pipeline secrets

To use pipeline secrets in Seqera Platform, the following extra IAM permissions must be provided: 
 
1. Create the AWS Batch [execution IAM role](https://docs.aws.amazon.com/batch/latest/userguide/execution-IAM-role.html#create-execution-role).

2. Add the `AmazonECSTaskExecutionRolePolicy` policy and the [Secrets policy execution role](secrets-policy-execution-role.json) to the execution IAM role created in step 1.

3. Specify the Execution role ARN in the **Batch execution role** field in the Seqera compute environment advanced settings.

4. Add the [Secrets policy instance role](secrets-policy-instance-role.json) to the ECS Instance role assigned to the Batch compute environment where your pipelines will be deployed. See [Amazon ECS instance role](https://docs.aws.amazon.com/batch/latest/userguide/instance_IAM_role.html) for more information.

5. Add the [Secrets policy](secrets-policy-account.json) to the IAM user or role used by Seqera to access your AWS account (specified in the Seqera credentials).

