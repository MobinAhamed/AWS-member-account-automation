# Deployment guide

This guide documents the AWS Management Console workflow shown in the supplied Word document, adapted to this repository's parameterized two-account template.

## 1. Prepare the template

1. Download `template.yaml` from this repository.
2. Do not put real account email addresses or company-specific account details into the public file.
3. The repository template uses parameters. You will provide account names and email addresses during stack creation.
4. Prepare two valid email addresses that are unique and not already associated with AWS accounts.

The original Word guide shows editing `AccountName` and `Email` directly in the template and saving the file. That is the workflow for the original hardcoded, single-account example. This repository version uses four parameters instead, so you do not need to edit the YAML for each deployment.

## 2. Sign in to AWS

1. Sign in to the AWS Management Console.
2. Use the AWS Organizations **management account**, or an approved deployment role in that account.
3. Confirm that the identity has `organizations:CreateAccount` and the CloudFormation permissions needed to create the stack.

## 3. Open CloudFormation and start a stack

1. Search for **CloudFormation** in the AWS Console and open the service.
2. Choose **Create stack**.
3. Select **With new resources (standard)**.

## 4. Upload the template

1. On the **Create stack** page, under **Prepare template**, select **Template is ready**.
2. Under **Specify template**, select **Upload a template file**.
3. Choose the `template.yaml` file and upload it.
4. Continue to the next step.

## 5. Specify stack details

1. Enter a stack name, for example `aws-org-member-account-provisioning`.
2. On the stack details page, provide:
   - `Account1Name`
   - `Account1Email`
   - `Account2Name`
   - `Account2Email`
3. Use valid account names and two unique email addresses. Do not reuse an email address already associated with an AWS account.
4. Choose **Next**.

## 6. Configure stack options and review

1. Review stack options and add tags if required by your organization.
2. The supplied template does not create IAM resources, so an IAM capability acknowledgement is not normally needed for this template. Follow any prompts shown by the console.
3. Choose **Next**.
4. Review all settings carefully before submitting the stack.

## 7. Create and monitor the stack

1. Submit the stack.
2. Monitor the CloudFormation **Events** tab.
3. Wait for the stack status to become `CREATE_COMPLETE`.
4. If the stack fails, inspect the first relevant failed event and its status reason before retrying. Account creation is asynchronous and may take time.

## 8. Verify the accounts

1. Open **AWS Organizations** in the management account.
2. Go to the accounts view and confirm that both new member accounts appear.
3. Check the account email inboxes for AWS messages.
4. Use the CloudFormation stack **Outputs** tab to retrieve the created account IDs.

## 9. Cleanup and lifecycle

The template sets `DeletionPolicy: Retain` on both account resources. Deleting the CloudFormation stack therefore retains the AWS member accounts; it does not close them. Manage retained accounts separately and deliberately.

## Security notes before publishing screenshots

The screenshots in the supplied Word document appear to show account email information and AWS Organizations/account identifiers. Do not publish those original screenshots in a public repository until you have redacted:
- Personal or company email addresses.
- AWS account IDs.
- Organization IDs.
- Any internal names, identifiers, or console details your employer does not authorize for public release.

Use synthetic example values in documentation. Never publish credentials, access keys, session tokens, or secrets.
