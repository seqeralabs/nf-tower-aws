# Seqera Forge IAM Policy for AWS

This directory contains the IAM policy required to use **Seqera Forge** with AWS. Seqera
Forge automates the creation and management of [AWS Batch](https://aws.amazon.com/batch/) compute
environments, simplifying your pipeline deployments. Instructions are available in the [Seqera
documentation](https://docs.seqera.io/platform-cloud/compute-envs/aws-batch#batch-forge) to
configure a IAM User for Batch Forge.

**If you are using Seqera Forge, you do not need the policies from the `launch/`
directory.**

## The `forge-policy.json` File

The [`forge-policy.json`](./forge-policy.json) file contains the necessary permissions for
Seqera Forge to operate. This policy is designed to be a general-purpose solution, but you
should review and customize it to meet your organization's security standards.

> [!WARNING]
> The default `forge-policy.json` grants broad permissions. We strongly recommend that you
> scope down these permissions to match your specific needs, as described in this document.

## How to Restrict Permissions

The `forge-policy.json` file is divided into several statements, each with a clear purpose. You can
restrict the permissions in each statement using resource-level restrictions, condition keys, and
resource tagging.

Below are examples of how to tighten the permissions for each section of the policy.

### Batch Execution

This section of the policy allows Seqera to manage Batch compute environments and jobs. You can
restrict these permissions to specific resources. e.g. by limiting to Job Queues and Compute
Environments starting with `TowerForge`, the [default JQ/CE prefix used by
Forge](https://docs.seqera.io/platform-enterprise/enterprise/configuration/overview#compute-environments).
You can also restrict permissions based on Resource tag (these need to be [set by users when setting
up a pipeline in Platform](https://docs.seqera.io/platform-enterprise/resource-labels/overview)).

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

Seqera Forge can create and manage the IAM roles needed for your pipelines. If you want to
restrict this, you can limit the role creation to a specific path or prefix. During Compute
Environment creation, you can optionally define pre-provisioned roles for your Compute Environment,
Head Job, and Instance profile, eliminating the need for these permissions.

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
> If you have a [custom
> prefix](https://docs.seqera.io/platform-enterprise/enterprise/configuration/overview#compute-environments)
> for your Seqera resources, replace `TowerForge-*` with your custom prefix.

### Pass Role to Batch

The `iam:PassRole` permission allows Seqera to pass IAM roles to AWS Batch. Permissions can be
restricted to only allow passing the roles created by Seqera Forge with the default prefix
`TowerForge-*` to the AWS Batch service:

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

### S3 Access

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
