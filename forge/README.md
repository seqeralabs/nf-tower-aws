# Seqera Platform Forge for AWS Batch

> [!WARNING]
> This example IAM policy is intended for use with Seqera Forge and provides a wide range of permissions to support common deployment scenarios. However, it may not be appropriate for all environments or security requirements. You should carefully review and tailor this policy to fit your organization's security standards and operational needs. See [IAM Policy Configuration & Tuning](#iam-policy-configuration--tuning) for more details.

Seqera Platform Forge automates the configuration of [AWS Batch](https://aws.amazon.com/batch/) compute environments and queues
required for the deployment of Nextflow pipelines.

To enable this feature, Seqera Platform requires the permissions listed in [this policy](forge-policy.json) file.

Attach the policy to the AWS IAM User associated to your Seqera configuration as described below:

1. Open the AWS [IAM console](https://console.aws.amazon.com/iam/home) and select **Users**.
1. Select or create the user associated with your Seqera configuration.
1. Select **Add inline policy**.
1. Select **JSON** and copy the content of the policy linked above.
1. Select **Review policy** and then **Create policy**.



## IAM Policy Configuration & Tuning

The example [Batch Forge Policy](forge-policy.json) provides comprehensive permissions for quick setup. However, you should review and adapt this policy to fit your specific security requirements.

### Restricting Permissions

This Readme explains how you can scope down the policy using:

1. Resource-level restrictions
2. AWS condition keys
3. Resource tagging

> [!NOTE]
> If you've configured a custom prefix for Compute Environments and IAM roles in your Seqera Platform Enterprise installation, remember to update the resource pattern accordingly.

### AWS Systems Manager (SSM)

Seqera Platform requires access to read from AWS Systems Manager (SSM) to [identify ECS Optimized AMI's](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/retrieve-ecs-optimized_AMI.html) for pipeline execution. If you don't need to retrieve any additional parameters as part of your workflow, you can restrict SSM access using a policy like the following:

```json
{
  "Sid": "FetchECSOptimizedAMIMetadata",
  "Effect": "Allow",
  "Action": "ssm:GetParameters",
  "Resource": ["arn:aws:ssm:*:<ACCOUNT_ID>:parameter/aws/service/ecs/*"]
}
```

### IAM Role Configuration

Seqera Platform Batch Forge by default creates and manages the lifecycle of IAM Roles & Policies used by Nextflow pipelines in Compute Environments. During Compute Environment creation, you can optionally define pre-provisioned roles for your Compute Environment, Head Job, and Instance profile, eliminating the need for these permissions.

If you want to allow Forge to manage IAM roles but restrict its permissions, you can use the following policy:

```json
{
  "Sid": "IAMRoleAndProfileManagement",
  "Effect": "Allow",
  "Action": [
    "iam:CreateInstanceProfile",
    "iam:DeleteInstanceProfile",
    "iam:AddRoleToInstanceProfile",
    "iam:RemoveRoleFromInstanceProfile",
    "iam:CreateRole",
    "iam:DeleteRole",
    "iam:GetRole",
    "iam:AttachRolePolicy",
    "iam:DetachRolePolicy",
    "iam:PutRolePolicy",
    "iam:DeleteRolePolicy",
    "iam:ListAttachedRolePolicies",
    "iam:ListRolePolicies",
    "iam:TagRole",
    "iam:TagInstanceProfile"
  ],
  "Resource": "arn:aws:iam::<ACCOUNT_ID>:role/TowerForge-*"
}
```

### PassRole

Seqera Batch Forge requires the ability to pass IAM roles to AWS services. The PassRole permission allows Forge to assign roles to AWS Batch services when creating compute environments.
You can restrict this permission to only allow passing roles with the appropriate prefix to the AWS Batch service:

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

### Batch Execution

Seqera Platform requires the ability to trigger workflows using AWS Batch when using it as a compute environment.
You can restrict this permission based on ARN or Resource tag (these need to be [set by users when setting up a pipeline in Platform](https://docs.seqera.io/platform-enterprise/resource-labels/overview)), like the following:

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

### Launch Template Management.

Seqera Platform requires the ability to create and manage EC2 launch templates using optimized AMIs identified via AWS Systems Manager (SSM).

> [!NOTE]
> AWS does not support restricting IAM permissions on EC2 launch templates based on specific
> resource names or tags. Because of this limitation, it is not currently possible to scope IAM
> permissions to specific launch templates. As a result, permission to operate on any resource `*`
> must be granted.

### S3 Data Access

Seqera Platform requires access to AWS S3 to list and inspect the contents of S3 buckets for Studios, DataExplorer and Identifying the Nextflow working directory.

This policy can be scoped down to only allow listing the buckets in the account along with limiting data retrieval to specific buckets.

```json
{
  "Sid": "S3ListBuckets",
  "Effect": "Allow",
  "Action": ["s3:ListAllMyBuckets"],
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
    "arn:aws:s3:::example-bucket",
    "arn:aws:s3:::example-bucket/*"
  ]
}
```

### Cloudwatch logs access

Seqera Platform requires access to CloudWatch logs in order to display relevant log data within the platform interface. The following permissions allow the platform to retrieve and query log events from the associated log group.

This policy can be scoped down to the specific log group used by the compute environment:

```json
{
  "Sid": "CloudWatchLogsAccess",
  "Effect": "Allow",
  "Action": [
    "logs:Describe*",
    "logs:Get*",
    "logs:List*",
    "logs:StartQuery",
    "logs:StopQuery",
    "logs:TestMetricFilter",
    "logs:FilterLogEvents"
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

NextFlow is capable of sending email reports from your Nextflow pipeline (such as MultiQC reports) via Amazon SES (Simple Email Service) permissions. You can restrict these permissions to specific sender and recipient addresses:

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
