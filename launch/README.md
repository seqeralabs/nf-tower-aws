# Tower Launch for AWS Batch

Nextflow Tower allows the seamless deployment of Nextflow data pipeles 
using [AWS Batch](https://aws.amazon.com/batch/) computing service. 

To enable such deployment create a AWS user account (or use an existing one)
granting at least the following IAM policies: 

- `AmazonS3ReadOnlyAccess` 
- `AmazonEC2ContainerRegistryReadOnly`
- `CloudWatchLogsReadOnlyAccess` 
- The following [custom policy](launch-policy.json) to grant the ability to submit and control Batch jobs.
- Grant write access to any S3 bucket used a pipeline work directory using the following [policy template](s3-bucket-write.json). 

