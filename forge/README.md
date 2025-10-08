# Seqera Batch Forge for AWS Batch

Seqera Platform can automate the configuration of [AWS Batch](https://aws.amazon.com/batch/) compute
environments and job queues for Nextflow pipelines using Batch Forge. As described in
the [introduction](../README.md), Forge will take care of creating AWS IAM Roles for each compute
environment, so the policies described in the [`launch/`](../launch) section are **not needed**.

To enable Batch Forge, the IAM user you configure in your Seqera Platform workspace requires the
permissions listed in the [`forge-policy.json`](forge-policy.json) file. Full instructions are
available in the [Seqera
documentation](https://docs.seqera.io/platform-cloud/compute-envs/aws-batch#batch-forge) to
configure a IAM User for Batch Forge.

> [!WARNING]
> The generic [`forge-policy.json`](forge-policy.json) IAM policy is intended for use with Seqera
> Forge only, and uses wide permissions to support common deployment scenarios.
> However, we recognize that it may not be appropriate for all environments or security
> requirements. You should carefully review and tailor the generic policy to fit your organization's
> security standards and operational needs, as described in this document.

## Restricting Forge Permissions

This Readme file explains how you can scope down the policy using:

1. Resource-level restrictions
2. AWS condition keys
3. Resource tagging

> [!NOTE]
> If you've configured a custom prefix for Compute Environments and IAM roles in your Seqera
> Platform Enterprise installation, remember to update `TowerForge-*` with the resource pattern
> you're using.

### Batch Execution

Seqera Platform requires the ability to trigger workflows using AWS Batch when using it as a compute
environment. You can restrict the `batch` actions to specific resources by replacing the `"Resource":
"*"` line with the ARN of your Batch job queues and compute environments, potentially using wildcards
to match multiple resources. You can also restrict permissions based on Resource tag (these need to
be [set by users when setting up a pipeline in
Platform](https://docs.seqera.io/platform-enterprise/resource-labels/overview)). For example:

```json
{
  "Sid": "BatchEnvironmentManagement",
  "Effect": "Allow",
  "Action": [
    "batch:CreateComputeEnvironment",
    "batch:CreateJobQueue",
    "batch:DeleteComputeEnvironment",
    "batch:DeleteJobQueue",
    "batch:DescribeComputeEnvironments",
    "batch:DescribeJobQueues",
    "batch:UpdateComputeEnvironment",
    "batch:UpdateJobQueue"
  ],
  "Resource": [
    "arn:aws:batch:<REGION>:<ACCOUNT_ID>:compute-environment/TowerForge-*",
    "arn:aws:batch:<REGION>:<ACCOUNT_ID>:job-queue/TowerForge-*"
  ],
  "Condition": {
    "StringEqualsIfExists": {
      "aws:ResourceTag/MyCustomTag": "MyCustomValue"
    }
  }
},
{
  "Sid": "BatchJobExecution",
  "Effect": "Allow",
  "Action": [
    "batch:CancelJob",
    "batch:DescribeJobDefinitions",
    "batch:DescribeJobs",
    "batch:ListJobs",
    "batch:RegisterJobDefinition",
    "batch:SubmitJob",
    "batch:TagResource",
    "batch:TerminateJob"
  ],
  "Resource": [
    "arn:aws:batch:<REGION>:<ACCOUNT_ID>:job-definition/*",
    "arn:aws:batch:<REGION>:<ACCOUNT_ID>:job/*"
  ],
  "Condition": {
    "StringEqualsIfExists": {
      "aws:ResourceTag/MyCustomTag": "MyCustomValue"
    }
  }
}
```

> [!WARNING]
> Restricting the `batch` actions using resource tags requires that you set the appropriate tags on
> each Seqera pipeline when configuring it in the Platform UI. Forgetting to set the tag will cause
> the pipeline to fail to run.

### Launch Template Management.

Seqera Platform requires the ability to create and manage EC2 launch templates using optimized AMIs identified via AWS Systems Manager (SSM).

> [!NOTE]
> AWS does not support restricting IAM permissions on EC2 launch templates based on specific
> resource names or tags. Because of this limitation, it is not currently possible to scope IAM
> permissions to specific launch templates. As a result, permission to operate on any resource `*`
> must be granted.

### AWS Systems Manager (SSM)

Seqera Platform requires access to read from AWS Systems Manager (SSM) to [identify ECS Optimized
AMI's](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/retrieve-ecs-optimized_AMI.html)
for pipeline execution.

### IAM Role Configuration

Seqera Platform Batch Forge by default creates and manages the lifecycle of IAM Roles & Policies
used by Nextflow pipelines in Compute Environments. During Compute Environment creation, you can
optionally define pre-provisioned roles for your Compute Environment, Head Job, and Instance
profile, eliminating the need for these permissions.

If you want to allow Forge to manage IAM roles but restrict the resources it can create to only
specific, you can use the following policy:

```json
{
  "Sid": "IAMRoleAndProfileManagement",
  "Effect": "Allow",
  "Action": [
    "iam:AddRoleToInstanceProfile",
    "iam:AttachRolePolicy",
    "iam:CreateInstanceProfile",
    "iam:CreateRole",
    "iam:DeleteInstanceProfile",
    "iam:DeleteRole",
    "iam:DeleteRolePolicy",
    "iam:DetachRolePolicy",
    "iam:GetRole",
    "iam:ListAttachedRolePolicies",
    "iam:ListRolePolicies",
    "iam:PutRolePolicy",
    "iam:RemoveRoleFromInstanceProfile",
    "iam:TagInstanceProfile",
    "iam:TagRole"
  ],
  "Resource": "arn:aws:iam::<ACCOUNT_ID>:role/TowerForge-*"
}
```

> [!NOTE]
> Adapt the resource definition if you configured your Seqera installation to create IAM roles with
> a different prefix than the default `TowerForge-*`.

### PassRole

Seqera requires the ability to `PassRole` to AWS Batch when using compute environments.
Permissions can be restricted to only allow passing the roles created by Seqera Forge with the
default prefix `TowerForge-*` to the AWS Batch service:

```json
{
  "Sid": "PassOnlyTowerForgeRolesToBatch",
  "Effect": "Allow",
  "Action": "iam:PassRole",
  "Resource": "arn:aws:iam::<ACCOUNT_ID>:role/TowerForge-*",
  "Condition": {
    "StringEquals": {
      "iam:PassedToService": "batch.amazonaws.com"
    }
  }
}
```

### S3 Data Access

Seqera Platform can list S3 buckets for
[Studios](https://docs.seqera.io/platform-cloud/studios/overview), [Data
Explorer](https://docs.seqera.io/platform-cloud/data/data-explorer) and to help identify the
Nextflow working directory.

The policy can be scoped down to only allow listing the buckets in the account along with limiting
data retrieval to specific buckets.

```json
{
  "Sid": "S3ListBuckets",
  "Effect": "Allow",
  "Action": "s3:ListAllMyBuckets",
  "Resource": "*"
},
{
  "Sid": "S3GetBucketData",
  "Effect": "Allow",
  "Action": [
    "s3:Get*",
    "s3:List*"
  ],
  "Resource": [
    "arn:aws:s3:::example-bucket1",
    "arn:aws:s3:::example-bucket1/*",
    "arn:aws:s3:::example-bucket2",
    "arn:aws:s3:::example-bucket2/*"
  ]
}
```

### Cloudwatch logs access

Seqera Platform requires access to CloudWatch logs to display relevant log data in the web
interface. The policy can be scoped down to limit access to the specific log group used by the
compute environment:

```json
{
  "Sid": "CloudWatchLogsAccess",
  "Effect": "Allow",
  "Action": [
    "logs:Describe*",
    "logs:FilterLogEvents",
    "logs:Get*",
    "logs:List*",
    "logs:StartQuery",
    "logs:StopQuery",
    "logs:TestMetricFilter"
  ],
  "Resource": "arn:aws:logs:<REGION>:<ACCOUNT_ID>:log-group:/aws/batch/job/*"
}
```

### FSx File Systems (optional)

Allow Forge to manage [AWS FSx file systems](https://aws.amazon.com/fsx/), if needed by the pipelines.

```json
{
  "Sid": "FSx",
  "Effect": "Allow",
  "Action": [
    "fsx:CreateFileSystem",
    "fsx:DeleteFileSystem",
    "fsx:DescribeFileSystems",
    "fsx:TagResource"
  ],
  "Resource": "*"
}
```

### EFS File Systems (optional)

Allow Forge to manage [AWS EFS file systems](https://aws.amazon.com/efs/), if needed by the pipelines.

```json
{
  "Sid": "EFS",
  "Effect": "Allow",
  "Action": [
    "elasticfilesystem:CreateFileSystem",
    "elasticfilesystem:DeleteFileSystem",
    "elasticfilesystem:CreateMountTarget",
    "elasticfilesystem:DeleteMountTarget",
    "elasticfilesystem:DescribeFileSystems",
    "elasticfilesystem:DescribeMountTargets",
    "elasticfilesystem:UpdateFileSystem",
    "elasticfilesystem:PutLifecycleConfiguration",
    "elasticfilesystem:TagResource"
  ],
  "Resource": "*"
}
```

### SES Policy (optional)

NextFlow is capable of sending email reports from your Nextflow pipeline (such as MultiQC reports)
via Amazon SES (Simple Email Service). The resource must be a wildcard to allow Platform to send
emails to any recipient. You can delete this statement if you don't want to use this feature, or you
can restrict these permissions to a specific sender and recipient addresses:

```json
{
  "Sid": "AllowEmailSendingFromSeqeraPlatform",
  "Effect": "Allow",
  "Action": "ses:SendRawEmail",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "ses:FromAddress": "seqera-platform-installation@example.com"
    },
    "ForAllValues:StringLike": {
      "ses:Recipients": ["*@example.com"]
    }
  }
}
```
