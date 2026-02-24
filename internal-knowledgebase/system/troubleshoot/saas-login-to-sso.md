---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "SaaS - Login to SSO"
summary: "In this article, we will show how to log in to AWS via SSO"
---
# SaaS - Login to SSO

## Overview

In this article, we will show how to log in to AWS via SSO

## Prerequisites

* AWS CLI version 2.0
* URL from AWS SSO

Note: The process has to be configured only once and then you can login in the future with the current profile name with the following command:
`# BROWSER=true aws sso login <profile_name>;`

## AWS Console

 ### Step 1

 You should have been provided a specific sign-in URL to the user portal

 ### Step 2

 Login to AWS SSO: &lt;https://exasol-ct.awsapps.com/start&gt;

 ### Step 3

 Then press **Enter**. We recommend that you bookmark this link to the portal now so that you can quickly access it later.

 ### Step 4

 Sign in using your user name and password. If you are prompted for a **Verification code**, check your email or MFA device and then copy and paste the code into the sign-in page.

### Step 5

Once signed in, you can access any AWS account and application that appears in the portal. Simply choose an icon.

Note:    Once you have been signed-in, your user portal session will be valid for 8 hours.

### AWS CLI

### Step 1

You can add an AWS SSO-enabled profile to your AWS CLI by running the following command, providing your AWS SSO start URL and the AWS Region that hosts the AWS SSO directory.

```sql
$ BROWSER=true aws configure sso
SSO start URL [None]: [None]: https://exasol-ct.awsapps.com/start
SSO region [None]:us-east-1
```

The AWS CLI attempts to open your default browser and begin the login process for your AWS SSO account.

If the AWS CLI cannot open the browser, the following message appears with instructions on how to manually start the login process.

Using a browser, open the following URL: https://exasol-ct.awsapps.com/start and enter the following code: XXXX-XXXX

Next, the AWS CLI displays the AWS accounts available for you to use. If you are authorized to use only one account, the AWS CLI selects that account for you automatically and skips the prompt. The AWS accounts that are available for you to use are determined by your user configuration in AWS SSO.

 Use the arrow keys to select the account you want to use with this profile. The "&gt;" character on the left points to the current choice. Press ENTER to make your selection.

 ### Step 2

Next, the AWS CLI confirms your account choice and displays the IAM roles that are available to you in the selected account. If the selected account lists only one role, the AWS CLI selects that role for you automatically and skips the prompt. The roles that are available for you to use are determined by your user configuration in AWS SSO.

 ### Step 3

Now you can finish the configuration of your profile, by specifying the [default output format](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html#cli-config-output), the [default AWS Region](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html#cli-config-region) &lt;eu-central-1&gt; to send commands to, and providing a [name for the profile](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-profiles) so you can reference this profile from among all those defined on the local computer.

```sql
Example:
  CLI default client Region [None]: eu-central-1<ENTER>
  CLI default output format [None]: json<ENTER>
  CLI profile name [Role_Name]: my-saas-profile<ENTER>
```

Note: If you specify default as the profile name, this profile becomes the one used whenever you run an AWS CLI command and do not specify a profile name.

## Additional References

AWS SSO Official: &lt;https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html&gt;
