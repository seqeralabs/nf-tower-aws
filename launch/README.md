# Launch-only required permissions

The policies in this directory are intended for use with Seqera Platform to allow IAM users to
launch pipelines with AWS Batch resources created manually. In certain environments, you may want or
you may be required to manually create and manage your [AWS Batch](https://aws.amazon.com/batch/)
resources.
Refer to the [Seqera
documentation](https://docs.seqera.io/platform-cloud/compute-envs/aws-batch#manual) for detailed
steps on how to manually configure your Batch environment.

As described in the [main README](../README.md), Seqera Platform can automate the AWS Batch
configuration steps through the use of [Batch Forge](../forge/).

> [!NOTE]
> Manual AWS Batch configuration is only necessary if you don't use Batch Forge.
Batch Forge automatically creates the AWS Batch queues required for your workflow executions.

> The policies in this section are **not needed** if you are using Batch Forge, as Forge will
> automatically create IAM Roles specifically for each Batch Compute Environment it creates and
> those roles will then be used to manage the resources within that environment. See the [Forge
> README](../forge/) for more information.

## Restricting Launch Permissions

The [launch policy](launch-policy.json) is the policy required by Seqera Platform to launch
pipelines using AWS Batch resources created manually. This policy must be attached to the IAM user
you specify in the credentials of your Seqera workspace.
The policy is divided into different statements to allow you to scope down the permissions.

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
    "arn:aws:batch:<REGION>:<ACCOUNT_ID>:compute-environment/MyManualCE",
    "arn:aws:batch:<REGION>:<ACCOUNT_ID>:job-queue/MyManualJQ"
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

### PassRole

Seqera requires the ability to `PassRole` to AWS Batch when using compute environments.
Permissions can be restricted to only allow passing the [execution IAM
roles](https://docs.aws.amazon.com/batch/latest/userguide/execution-IAM-role.html#create-execution-role)
you created manually to the AWS Batch service:

```json
{
  "Sid": "PassRolesToBatch",
  "Effect": "Allow",
  "Action": "iam:PassRole",
  "Resource": "arn:aws:iam::<ACCOUNT_ID>:role/MyExecutionRole",
  "Condition": {
    "StringEquals": {
      "iam:PassedToService": "batch.amazonaws.com"
    }
  }
}
```

### CloudWatch Logs Access

Seqera Platform requires access to CloudWatch logs to display relevant log data in the web
interface. The policy can be scoped down to limit access to the specific log group used by the
compute environment:

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

### Pipeline Secrets (optional)

[Pipeline Secrets](https://docs.seqera.io/platform-cloud/secrets/overview) require additional
permissions on the IAM User. The listing of secrets cannot be restricted, but the management actions
can be restricted to only allow managing secrets in a specific region and account. Note that Seqera
only creates secrets with the `tower-` prefix. The region must be the same region where the pipeline
runs.

```json
{
  "Sid": "OptionalPipelineSecretsListing",
  "Effect": "Allow",
  "Action": "secretsmanager:ListSecrets",
  "Resource": "*"
},
{
  "Sid": "OptionalPipelineSecretsManagementCanBeRestricted",
  "Effect": "Allow",
  "Action": [
    "secretsmanager:DescribeSecret",
    "secretsmanager:DeleteSecret",
    "secretsmanager:CreateSecret"
  ],
  "Resource": "arn:aws:secretsmanager:<REGION>:<ACCOUNT_ID>:secret:tower-*"
}
```

#### Additional steps required to use secrets in a pipeline

To successfully use pipeline secrets, you must also:

1. Create a [AWS Batch Execution IAM
   role](https://docs.aws.amazon.com/batch/latest/userguide/execution-IAM-role.html#create-execution-role)
   with the
   [`AmazonECSTaskExecutionRolePolicy`](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_execution_IAM_role.html)
   AWS managed policy attached to it, and an inline policy to allow fetching Secrets Manager
   secrets:

   ```json
   {
     "Effect": "Allow",
     "Action": "secretsmanager:GetSecretValue",
     "Resource": "arn:aws:secretsmanager:*:*:secret:tower-*"
   }
   ```

1. Specify the Execution IAM role ARN in the **Batch execution role** field in the Seqera compute environment advanced settings.

1. Edit and attach the following policy to the ECS Instance role assigned to the Batch compute
   environment used by your Seqera pipelines. See [Amazon ECS instance
   role](https://docs.aws.amazon.com/batch/latest/userguide/instance_IAM_role.html) for more
   information.

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": "secretsmanager:ListSecrets",
         "Resource": "*"
       },
       {
         "Effect": "Allow",
         "Action": "secretsmanager:GetSecretValue",
         "Resource": "arn:aws:secretsmanager:*:*:secret:tower-*"
       },
       {
         "Effect": "Allow",
         "Action": [
           "iam:GetRole",
           "iam:PassRole"
         ],
         "Resource": "arn:aws:iam::<ACCOUNT_ID>:role/<EXECUTION-ROLE-NAME>"
       }
     ]
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

## Seqera role trust policy (optional)

You can optionally create a Seqera role trust policy to allow EC2 instances or EKS clusters (depending on your Seqera deployment) to assume the Seqera IAM role.

1. Download the [Seqera role trust policy](seqera-role-trust-policy.json).
1. Replace `<ACCOUNT_ID>` with your AWS account ID.
1. Replace `<USER_NAME>` and/or `<ROLE_NAME>` with the users and or roles that must be able to assume the Seqera IAM role.
