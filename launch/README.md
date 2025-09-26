# Permissions to launch pipelines with manually created AWS Batch resources

In certain environments, you may want or be required to manually create and manage your [AWS
Batch](https://aws.amazon.com/batch/) resources. Refer to the [Seqera
documentation](https://docs.seqera.io/platform-cloud/compute-envs/aws-batch#manual) for all required
steps to configure your environment.

As described in the [introduction](../README.md), Seqera Platform can automate the AWS Batch
configuration steps by using [Batch Forge](../forge/).

> [!NOTE]
> The policies in this section are **not needed** if you are using Batch Forge, as Forge will
> automatically create IAM Roles specifically for each Batch Compute Environment it creates and
> those roles will then be used to manage the resources within that environment. See the [Forge
> README](../forge/) for more information.

## IAM policies to launch pipelines

The policies in this directory are intended for use with Seqera Platform when not using Batch Forge.
Not all permissions may be needed depending on your specific setup.

### Seqera role trust policy (optional)

You can optionally create a Seqera role trust policy to allow EC2 instances or EKS clusters (depending on your Seqera deployment) to assume the Seqera IAM role.

1. Download the [Seqera role trust policy](seqera-role-trust-policy.json).
1. Replace `<ACCOUNT_ID>` with your AWS account ID.
1. Replace `<USER_NAME>` and/or `<ROLE_NAME>` with the users and or roles that must be able to assume the Seqera IAM role.

### Pipeline secrets

To use pipeline secrets in Seqera Platform, the following extra IAM permissions must be provided:

1. Create the AWS Batch [execution IAM role](https://docs.aws.amazon.com/batch/latest/userguide/execution-IAM-role.html#create-execution-role).

2. Add the `AmazonECSTaskExecutionRolePolicy` policy and the [Secrets policy execution role](secrets-policy-execution-role.json) to the execution IAM role created in step 1.

3. Specify the Execution role ARN in the **Batch execution role** field in the Seqera compute environment advanced settings.

4. Add the [Secrets policy instance role](secrets-policy-instance-role.json) to the ECS Instance role assigned to the Batch compute environment where your pipelines will be deployed. See [Amazon ECS instance role](https://docs.aws.amazon.com/batch/latest/userguide/instance_IAM_role.html) for more information.

5. Add the [Secrets policy](secrets-policy-account.json) to the IAM user or role used by Seqera to access your AWS account (specified in the Seqera credentials).
