# AWS Cloud ComputeEnvironment IAM permission

To configure a Seqera AWS Cloud compute environment, AWS credentials need to be provisioned beforehand in your AWS account for the compute environment creation and pipeline launch to work. The CloudFormation template in [template.yaml](template.yaml) can be used to provision such resources, or you could manually create the user and assign the policies you can find in the [policy.json](policy.json) file.

> [!NOTE]
> These permission give start access to resources to cover most of the use cases. You can restrict those according to resource naming or tag conditons depending on your conventions.

## Cloudformation template

### Created resources

The template will deploy the following resources:
- An IAM user
- Access keys for the created user
- An IAM Role that can be assumed by the created user, with IAM Inline Policies attached to grant enough permissions

### Accepted parameters

The following parameters are accepted to customise the names of the resources created:

- IAMUserName: will define the name of the IAM user created, defaults to `SeqeraPlatform`.
- RoleName: will define the name of the IAM role to be assumed, defaults to `SeqeraPlatformRole`.

### Generated Output

The stack will make available as output the following information:

- SeqeraPlatformUserAccessKeyId: the AWS Access Key for the created user.
- SeqeraPlatformUserSecretAccessKey: the AWS Secret Access Key for the created user.
- SeqeraPlatformRole: the IAM role to be assumed by the user to work with the AWS Cloud CE.

## Usage

There are 2 ways to provision resources using this template: through the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html) or through the [AWS console](https://aws.amazon.com/console/).

### Usage with AWS CLI

To create the stack through the AWS CLI, download the file [template.yaml](template.yaml) on your machine, and from the same directory run:

```bash
aws cloudformation create-stack --stack-name seqera-aws-cloud-bootstrap --template-body file://template.yaml --capabilities CAPABILITY_NAMED_IAM
```

> [!NOTE]
> --capabilities CAPABILITY_NAMED_IAM is necessary to acknowledge the fact this template will create and manage IAM resources.

If you want to customise any of the parameters, you can pass them to the aws cli command following its convetion. For example to customise the IAM user name and IAM role name you can create the stack with the following:

```bash
aws cloudformation create-stack --stack-name seqera-aws-cloud-bootstrap --template-body file://template.yaml \
    --capabilities CAPABILITY_NAMED_IAM \
    --parameters ParameterKey=IAMUserName,ParameterValue=MyCustomIAMUserName ParameterKey=RoleName,ParameterValue=MyCustomIAMRoleName
```

If you wish to make any modification to the resources after they are created, you can run the update command after modifying the template

```bash
aws cloudformation update-stack --stack-name seqera-aws-cloud-bootstrap --template-body file://template.yaml --capabilities CAPABILITY_NAMED_IAM
```

To see the output in your console you can run

```bash
aws cloudformation describe-stacks --stack-name seqera-aws-cloud-bootstrap --query "Stacks[0].Outputs"
```

To delete the resources created with this stack, you can run:

```bash
aws cloudformation delete-stack --stack-name seqera-aws-cloud-bootstrap
```

### Usage with AWS Console

You can imprort the template directly in the AWS Console if you're not comfortable with the aws cli. AWS has extensive documentation on how to do [Create a stack from the CloudFormation console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html).

## JSON IAM policy

If you don't feel comfortable managing this through CloudFormation, you can find a policy expressed in JSON format in [policy.json] that you can attach manually to a role or a user.
