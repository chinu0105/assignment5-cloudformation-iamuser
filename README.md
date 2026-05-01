# assignment5-cloudformation-iamuser

This repository contains an AWS CloudFormation template for creating a contractor IAM user, attaching that user to a Contractors group, and granting the group scoped access to EC2, S3, and EKS.

## Files

- `iam-contractors.yml` — CloudFormation template defining:
  - IAM Managed Policy for contractor access
  - IAM Group named `Contractors`
  - IAM User created from the `IAMUserName` parameter
  - Outputs for user name, user ARN, group name, and policy ARN

## Purpose

The template is designed to support an environment where contractors need controlled access to AWS resources without the ability to escalate IAM privileges.

## Resources created

- `AWS::IAM::ManagedPolicy` — `ContractorsAccessPolicy`
- `AWS::IAM::Group` — `Contractors`
- `AWS::IAM::User` — user created with the provided `IAMUserName`

## Permissions granted

The contractors group policy includes:

- EC2: Describe, list, start, and stop instance operations
- S3: List buckets, list objects, get/put/delete objects
- EKS: Describe clusters and nodegroups, access Kubernetes API

It also includes a deny statement for IAM privilege escalation actions such as `iam:CreateUser`, `iam:DeleteUser`, `iam:AttachUserPolicy`, and `iam:PassRole`.

## Deployment

Use the AWS CLI or AWS CloudFormation console to deploy the stack.

Example using AWS CLI:

```bash
aws cloudformation deploy \
  --template-file iam-contractors.yml \
  --stack-name contractors-iam-stack \
  --parameter-overrides IAMUserName=contractor-user Environment=dev \
  --capabilities CAPABILITY_NAMED_IAM
```

## Parameters

- `IAMUserName` — Name of the IAM user to create. Default: `contractor-user`
- `Environment` — Deployment environment tag. Allowed values: `dev`, `staging`, `prod`. Default: `dev`

## Outputs

- `IAMUserName` — IAM user name created by the stack
- `IAMUserArn` — ARN of the created IAM user
- `GroupName` — Group name assigned to the user
- `PolicyArn` — ARN of the managed policy attached to the group

## Notes

- The policy uses wildcard resources (`'*'`) for simplicity. Review and tighten resource scopes before production use.
- The template tags the user with `ManagedBy=CloudFormation`, `Environment`, and `Purpose=ContractorAccess`.
