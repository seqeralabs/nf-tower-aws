# AWS IAM policies for Seqera Platform

This repo contains the policies that need to be set on AWS IAM Users to allow Seqera Platform to
operate properly with AWS.

## Difference between `launch` and `forge` policies

The policies in the [`launch/`](../launch/) directory are needed if you manually create and manage
AWS Batch compute environments and queues. Instructions are available in the [Seqera
documentation](https://docs.seqera.io/platform-cloud/compute-envs/aws-batch#manual) to manually set
up all required resources for a pipeline to complete successfully.

The policies in the [`forge/`](../forge/) directory are needed if you want to let Seqera
automatically create and manage AWS Batch compute environments and queues for you, using the Batch
Forge feature. Instructions are available in the [Seqera
documentation](https://docs.seqera.io/platform-cloud/compute-envs/aws-batch#batch-forge) to
configure a IAM User for Batch Forge.
When letting Batch Forge handle the AWS Batch resources, you do not need the policies in the
`launch/` directory, as Forge will automatically create IAM Roles specifically for each Batch
Compute Environment it creates and those roles will then be used to manage the resources within
that environment.
