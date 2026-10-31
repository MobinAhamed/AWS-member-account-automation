# AWS Organizations Account Provisioning with CloudFormation

A CloudFormation template that provisions **two AWS member accounts** inside an existing AWS Organization. It uses the native `AWS::Organizations::Account` resource and creates the accounts sequentially.

> **Important:** This repository contains a two-account example. It is not a dynamic account factory and does not accept an arbitrary list of accounts.

## Features

- Creates two member accounts in an existing AWS Organization.
- Accepts account names and unique email addresses as CloudFormation parameters.
- Uses `DependsOn` so the second account is created after the first.
- Uses `DeletionPolicy: Retain` so deleting the CloudFormation stack does not delete/close the created accounts.
- Exposes the created account IDs as stack outputs.

## Architecture

1. An administrator deploys `template.yaml` from the AWS Organizations management account.
2. CloudFormation creates the first member account.
3. `DependsOn` makes the second account wait for the first resource.
4. CloudFormation returns the account IDs in stack outputs.

See [deployment-guide.md](deployment-guide.md).

## Prerequisites

- An existing AWS Organization.
- Deploy using credentials for the **management account**. The `AWS::Organizations::Account` resource can only be created from the organization's management account.
- An IAM identity with permission to create accounts (`organizations:CreateAccount`) and the CloudFormation permissions needed to create and manage the stack.
- Two valid email addresses, each not already associated with another AWS account.
- AWS CLI configured, or access to the CloudFormation console.

## Deploy using the AWS Management Console

The original step-by-step guide supplied with this project uses the CloudFormation console. The screenshots show this workflow:

1. Edit the template's account name and unique account email address, then save the template file.
2. Sign in to the AWS Management Console.
3. Open **CloudFormation** and choose **Create stack** → **With new resources (standard)**.
4. Under **Prepare template**, select **Template is ready**.
5. Under **Specify template**, choose **Upload a template file**, select the saved YAML file, and upload it.
6. Enter a stack name and choose **Next**.
7. Review **Configure stack options**. Add tags or IAM capabilities only when required by the template and your organization's deployment process. The supplied template does not define IAM resources.
8. Choose **Next**, review the settings, acknowledge any required capabilities, and submit the stack.
9. Wait for the stack status to become `CREATE_COMPLETE`.
10. Open **AWS Organizations** and verify the new member accounts. Check the account email inbox for AWS messages.

**For this repository's parameterized template:** the console asks for `Account1Name`, `Account1Email`, `Account2Name`, and `Account2Email` on the **Specify stack details** page. Enter two valid, unique email addresses. Do not hardcode real email addresses into the public template.

See [architecture.md](architecture.md).

## Deploy using the AWS CLI

Validate the template:

```bash
aws cloudformation validate-template \
  --template-body file://template.yaml
```

Deploy the stack (replace the example values with real account names and unique email addresses):

```bash
aws cloudformation deploy \
  --template-file template.yaml \
  --stack-name aws-org-member-account-provisioning \
  --parameter-overrides \
    Account1Name=Development-Account \
    Account1Email=aws-dev@example.com \
    Account2Name=Testing-Account \
    Account2Email=aws-test@example.com
```

Do not use the example email addresses as real account emails. Supply addresses you control and that meet AWS account email requirements.

Check stack status and outputs:

```bash
aws cloudformation describe-stacks \
  --stack-name aws-org-member-account-provisioning
```

Account creation is asynchronous. A successful CloudFormation request does not necessarily mean the new account is immediately ready to use. Wait for the stack operation to finish and check the resulting account state in AWS Organizations.

## Security considerations

- Never commit AWS access keys, secret keys, session tokens, or passwords.
- Do not publish real account email addresses, internal account IDs, organization IDs, or other company-specific information without authorization.
- Use a dedicated deployment role and least-privilege permissions. Avoid long-lived IAM user access keys.
- Protect access to the Organizations management account. `organizations:CreateAccount` is a sensitive permission because it can create accounts and associated costs.
- AWS Organizations normally creates an `OrganizationAccountAccessRole` in each new account unless a different role name is specified. This role is highly privileged; tightly control who can assume it.
- Review the account's billing, contacts, security baseline, logging, and guardrails after creation.
- `DeletionPolicy: Retain` means stack deletion leaves the member accounts in the organization. It does not close them.

## Important AWS behavior and limitations

- `AWS::Organizations::Account` can only be used from the Organizations management account.
- AWS account creation is asynchronous.
- Multiple account resources in one template must be sequenced with `DependsOn`; simultaneous creation can fail.
- Account names and email addresses cannot be changed through a CloudFormation stack update for this resource.
- Each account email must be valid and unique to an AWS account.
- Account quotas and account-creation rate limits apply.
- Creating an account does not automatically configure a full landing zone, security controls, IAM Identity Center access, logging, or workload infrastructure.

## Repository contents

```text
.
├── template.yaml
├── README.md
├── LICENSE
└── docs/
    ├── architecture.md
    └── deployment-guide.md
```

## Disclaimer

This is a community example, provided as-is. Test in a controlled AWS environment before production use. Review AWS documentation and your organization's security and account-governance requirements.

## References

- [AWS CloudFormation: AWS::Organizations::Account](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-organizations-account.html)
- [AWS Organizations: CreateAccount API](https://docs.aws.amazon.com/accounts/latest/reference/API_CreateAccount.html)
- [AWS Organizations quotas](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_reference_limits.html)
