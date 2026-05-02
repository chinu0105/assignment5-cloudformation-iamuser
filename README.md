# assignment5-cloudformation-iamuser

This repository contains an AWS CloudFormation template that creates a contractor IAM user, places the user in a `Contractors` group, and attaches scoped permissions for EC2, S3, and EKS.

## Files

- `iam-contractors.yml` — CloudFormation template that defines:
  - `AWS::IAM::ManagedPolicy` for contractor permissions
  - `AWS::IAM::Group` named `Contractors`
  - `AWS::IAM::User` created from the `IAMUserName` parameter
  - Outputs for user name, user ARN, group name, and policy ARN

## Purpose

The template provides a controlled contractor access model by granting necessary AWS permissions while explicitly denying IAM privilege escalation actions.

## Resources created

- `AWS::IAM::ManagedPolicy` — `ContractorsAccessPolicy`
- `AWS::IAM::Group` — `Contractors`
- `AWS::IAM::User` — created from the `IAMUserName` parameter

## Permissions granted

The managed policy attached to the `Contractors` group includes:

- EC2: `Describe*`, `List*`, `StartInstances`, `StopInstances`
- S3: `ListAllMyBuckets`, `ListBucket`, `GetObject`, `PutObject`, `DeleteObject`
- EKS: `DescribeCluster`, `ListClusters`, `ListNodegroups`, `DescribeNodegroup`, `AccessKubernetesApi`

It also includes a deny statement for IAM escalation actions such as:

- `iam:CreateUser`
- `iam:DeleteUser`
- `iam:AttachUserPolicy`
- `iam:PassRole`

## Deployment

Deploy the stack with the AWS CLI or CloudFormation console.

Example:

```bash
aws cloudformation deploy \
  --template-file iam-contractors.yml \
  --stack-name contractors-iam-stack \
  --parameter-overrides IAMUserName=contractor-user Environment=dev \
  --capabilities CAPABILITY_NAMED_IAM
```

> Ensure your AWS credentials are configured before deploying. This repository does not include AWS credentials or secrets.

## Parameters

- `IAMUserName` — IAM user name to create. Default: `contractor-user`
- `Environment` — Environment tag. Allowed values: `dev`, `staging`, `prod`. Default: `dev`

## Outputs

- `IAMUserName` — IAM user name created by the stack
- `IAMUserArn` — ARN of the created IAM user
- `GroupName` — Name of the group assigned to the user
- `PolicyArn` — ARN of the managed policy attached to the group

## Notes

- The current policy uses wildcard resources (`'*'`) for simplicity. Tighten resource scope for production deployments.
- The created user is tagged with `ManagedBy=CloudFormation`, the selected `Environment`, and `Purpose=ContractorAccess`.

