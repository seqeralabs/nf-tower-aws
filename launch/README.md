# Seqera Launch IAM Policy for AWS Batch

This directory contains the IAM policy to allow Seqera Platform to submit pipelines to
[AWS Batch](https://aws.amazon.com/batch/) using **manually managed Batch resources**. This setup
is recommended to users who want or need to create and manage their own compute environments and
queues. Refer to the [Seqera 
documentation](https://docs.seqera.io/platform-cloud/compute-envs/aws-batch#manual) for detailed
steps on how to manually configure your Batch environment.

**If you want to use Seqera Forge to automatically manage your AWS resources, you do not need
this policy.** See the [`forge/`](../forge) directory for the correct policy.

## The `launch-policy.json` File

The [`launch-policy.json`](./launch-policy.json) file contains the permissions that Seqera
Platform needs to launch pipelines using your existing AWS Batch infrastructure. As with the `forge`
policy, you should review and customize this policy to fit your security requirements.

> [!WARNING]
> The default `launch-policy.json` grants broad permissions. We strongly recommend that you scope
> down these permissions to match your specific needs, as described in this document.

## How to Restrict Permissions

The `launch-policy.json` file is divided into several statements, each with a clear purpose. You can
restrict the permissions in each statement using resource-level restrictions, condition keys, and
resource tagging, or by dropping certain actions completely if they are not needed for your use
case.

Below are examples of how to tighten the permissions for each section of the policy.

### AWS Batch Management

This section of the policy allows Seqera to manage Batch compute environments and jobs. You can
restrict these permissions to specific resources, e.g. by limiting access to the Job Queues and
Compute Environments created manually. You can also restrict permissions based on Resource tag
(these need to be set by users when [setting up a pipeline in
Platform](https://docs.seqera.io/platform-enterprise/resource-labels/overview)).

```json
{
  "Sid": "BatchEnvironmentManagement",
  "Effect": "Allow",
  "Action": [
    "batch:DescribeComputeEnvironments",
    "batch:DescribeJobQueues"
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
  "Sid": "BatchJobsManagement",
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

### Pass Role to Batch

The `iam:PassRole` permission allows Seqera to pass [execution IAM
roles](https://docs.aws.amazon.com/batch/latest/userguide/execution-IAM-role.html#create-execution-role)
to AWS Batch. Permissions can be restricted to only allow passing specific roles you create to the
AWS Batch and EC2 services:

```json
{
  "Sid": "PassRolesToBatch",
  "Effect": "Allow",
  "Action": "iam:PassRole",
  "Resource": "arn:aws:iam::<ACCOUNT_ID>:role/MyExecutionRole",
  "Condition": {
    "StringEquals": {
      "iam:PassedToService": [
        "batch.amazonaws.com",
        "ec2.amazonaws.com"
      ]
    }
  }
}
```

### CloudWatch Logs Access

Seqera Platform requires access to CloudWatch logs to display relevant log data in the web
interface. The policy can be scoped down to limit access to the [specific log
group](https://docs.seqera.io/platform-cloud/compute-envs/aws-batch#advanced-options) defined on the
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

### S3 Access (optional)

Seqera Platform offers several products to manipulate data on AWS S3 buckets via its UI, like
[Studios](https://docs.seqera.io/platform-cloud/studios/overview) and [Data
Explorer](https://docs.seqera.io/platform-cloud/data/data-explorer). To improve the user experience,
Platform automatically fetches the list of buckets the user has access to, and
provides the list in a dropdown menu to be used as Nextflow working directory.
The Studios and Data Explorer features are optional, and users can type the bucket name manually.

The policy can be scoped down to allow listing all the buckets in the account (necessary to populate
the dropdown menu in the UI), and to allow limited Read/Write permissions in certain S3 buckets used
by Studios/Data Explorer.

```json
{
  "Sid": "S3ListBuckets",
  "Effect": "Allow",
  "Action": "s3:ListAllMyBuckets",
  "Resource": "*"
},
{
  "Sid": "S3ReadWriteBucketsForStudiosDataExplorer",
  "Effect": "Allow",
  "Action": [
    "s3:Get*",
    "s3:List*",
    "s3:PutObject"
  ],
  "Resource": [
    "arn:aws:s3:::example-bucket-read-write-studios",
    "arn:aws:s3:::example-bucket-read-write-studios/*",
    "arn:aws:s3:::example-bucket-read-write-data-explorer",
    "arn:aws:s3:::example-bucket-read-write-data-explorer/*"
  ]
}
```

### Pipeline Secrets (optional)

Platform can synchronize the [Pipeline
Secrets](https://docs.seqera.io/platform-cloud/secrets/overview) defined on the Platform workspace
with AWS Secrets Manager, which requires additional permissions on the IAM User.

The listing of secrets cannot be restricted, but the management actions can be restricted to only
allow managing secrets in a specific account and region, which must be the same region where the
pipeline runs. Note that Seqera only creates secrets with the `tower-` prefix.

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

To successfully use pipeline secrets, the IAM Roles manually created must follow the steps detailed
in the [Seqera
documentation](https://docs.seqera.io/platform-cloud/secrets/overview#aws-secrets-manager-integration).
