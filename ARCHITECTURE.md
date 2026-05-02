# Architecture — CloudFormation IAM User Pipeline

## Template Design Decisions

## Decision 1 — Privilege Escalation DENY
The ManagedPolicy includes an explicit DENY on:
- iam:CreatePolicy
- iam:AttachUserPolicy  
- iam:PutUserPolicy

This prevents a contractor from escalating their own
privileges — a common IAM attack vector in AWS.
In my EKS platform at Siemens I enforce 25 similar 
IRSA/SGP validation rules as mandatory CI gates.

## Decision 2 — Parameterisation
IAMUserName and Environment are Parameters, not 
hardcoded values. The same template deploys to 
dev/staging/prod by changing one input — 
no template duplication.

## Decision 3 — cfn-lint Before Deploy
The validate job runs cfn-lint on the template before
any deployment attempt. This catches:
- Invalid resource types
- Incorrect property names  
- Missing required fields
- IAM policy syntax errors

catch errors before AWS API 
calls, not after a half-deployed stack.

## Decision 4 — Path-based Trigger
Pipeline only fires when iam-contractors.yml changes.
Committing unrelated files doesn't waste CI minutes
or risk accidental redeployment.

## Pipeline Flow
template change → cfn-lint validate → 
aws cloudformation deploy → describe outputs → 
Job Summary

## Why No Live Deployment
No personal AWS account available for this demo.
In production I would use OIDC (not static keys) —
GitHub Actions assumes an IAM role via web identity
federation. No long-lived credentials stored anywhere.

## What I Would Add in Production
- Stack drift detection on a schedule
- CloudTrail alerting on stack changes
- AWS Config rules validating IAM compliance
- Separate stacks per environment with promotion gates
