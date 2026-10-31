# Architecture: AWS Organizations Account Provisioning

## Purpose

This project uses AWS CloudFormation to create two member accounts in an existing AWS Organization.

## Components

- **Administrator:** Supplies the account names and unique email addresses and initiates deployment.
- **AWS CloudFormation:** Reads the template, creates the account resources, tracks stack operations, and exposes account IDs as outputs.
- **AWS Organizations:** Processes the account-creation requests and adds the new accounts to the organization.
- **Member accounts:** The two accounts created by the template.

## Provisioning sequence

```text
Administrator
     |
     | Deploy template.yaml with parameters
     v
AWS CloudFormation
     |
     | Create ChildAccount1
     v
AWS Organizations
     |
     | ChildAccount1 completes
     v
CloudFormation DependsOn
     |
     | Create ChildAccount2
     v
AWS Organizations
     |
     v
Two member accounts in the organization
```

`ChildAccount2` declares `DependsOn: ChildAccount1`. This is important because AWS documents that multiple `AWS::Organizations::Account` resources in one template must be created sequentially.

## Trust and permissions boundary

Deployment must originate from the AWS Organizations management account. The deploying IAM principal requires `organizations:CreateAccount` and the CloudFormation permissions required for the stack operation. Use a dedicated role and least-privilege policies; do not use root credentials or commit credentials to the repository.

AWS Organizations normally provisions an `OrganizationAccountAccessRole` in each new member account. This role grants administrative access from the management account when assumed by authorized principals. Treat it as privileged access and restrict role assumption.

## Lifecycle and deletion behavior

Each account resource uses `DeletionPolicy: Retain`. Deleting the CloudFormation stack therefore retains the account resource rather than closing the member account. Account lifecycle and closure must be managed deliberately through AWS Organizations and in accordance with AWS requirements.

## Asynchronous operation

Account creation is asynchronous. A successful initial request does not prove that account initialization is complete. Monitor the CloudFormation stack operation and verify account state in AWS Organizations. CloudTrail can also be used to review account-creation events.

## Out of scope

This template does not configure:
- Organizational units or move accounts into OUs.
- Service Control Policies (SCPs).
- AWS IAM Identity Center users, groups, or permission sets.
- Centralized CloudTrail, AWS Config, or security services.
- VPCs, networking, workloads, or account baselines.
- A dynamic, arbitrary number of accounts.


## Relationship to the supplied walkthrough

The supplied Word document (three pages) documents a console-driven deployment:
- Page 1: edit account name/email in the template, open CloudFormation, and create a new stack.
- Page 2: upload the template, specify the stack name, configure options, and wait for stack creation to complete.
- Page 3: verify the account in AWS Organizations and check the account email inbox.

The original walkthrough screenshot shows a single account resource with values entered directly in the YAML. The current repository template is a parameterized two-account version, so the console workflow is similar but the four account parameters are entered on the stack details page.
