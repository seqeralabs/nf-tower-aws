# Tower Launch for AWS Batch

Nextflow Tower allows the seamless deployment of Nextflow data pipelines 
using [AWS Batch](https://aws.amazon.com/batch/) computing service. 

To enable such deployment create a AWS user account (or use an existing one)
granting at least the following IAM permission: 

- `AmazonS3ReadOnlyAccess` 
- `AmazonEC2ContainerRegistryReadOnly`
- `CloudWatchLogsReadOnlyAccess` 
- The following [custom policy](launch-policy.json) to grant the ability to submit and control Batch jobs.
- Grant write access to any S3 bucket used a pipeline work directory using the following [policy template](s3-bucket-write.json). 

You will also need to create the AWS Batch queue(s) required to deploy the Nextflow execution. 

Tower can automate this configuration step, granting the required permissions to it as described 
at [this link](../forge/README.md).


### Pipeline Secrets

If you are planning to use the Pipeline Secrets feature provided by Tower, the following
extra IAM permissions should be provided: 
 
1. Create the AWS Batch [IAM Execution role](https://docs.aws.amazon.com/batch/latest/userguide/execution-IAM-role.html#create-execution-role) as specified in the AWS documentation.

2. Make sure to add the *Execution role* created above the `AmazonECSTaskExecutionRolePolicy` policy
  and the custom policy at [this link](secrets-policy-execution-role.json).

3. Specify the *Execution role* ARN in the field "Batch execution role" in the Tower compute environment creation
  advanced settings.

4. Add the custom policy at [this link](secrets-policy-instance-role.json) to the ECS Instance role
  associated to the Batch compute environment that's going to be used to deploy your pipelines.
  Find more details about the Instance role at [this link](https://docs.aws.amazon.com/batch/latest/userguide/instance_IAM_role.html).

5. Add the custom policy at [this link](secrets-policy-account.json) to the IAM user or role granting
  access to your AWS account to Tower (the one specified in the Tower credentials).

