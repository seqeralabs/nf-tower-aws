# Tower Forge for AWS Batch

Tower Forge automates the configuration of [AWS Batch](https://aws.amazon.com/batch/) compute environments and queues
required for the deployment of Nextflow pipelines. 

To enable this feature Tower require the permissions listed in [this policy](forge-policy.json) file. 

Attach the policy to the AWS user account associatedto your Tower configuration, as described below: 

1) Open the AWS [IAM console](https://console.aws.amazon.com/iam/home)
2) Select *Users* on the left menu 
3) Choose (or create) the user associated in your Tower configuration 
4) Click on *Add inline policy*
5) Choose *JSON* and copy the content of the policy linked above. 
6) Click on the button *Review policy* and confirm the operation clicking *Create policy* 

