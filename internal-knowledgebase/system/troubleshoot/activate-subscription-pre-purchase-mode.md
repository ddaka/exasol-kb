---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Activating Subscription to Pre-Purchase Mode"
summary: "This playbook provides step-by-step instructions for the operations team to activate a subscription to pre-purchase mode. Follow these steps carefully to ensure successful..."
---
# Activating Subscription to Pre-Purchase Mode

## Purpose

This playbook provides step-by-step instructions for the operations team to activate a subscription to pre-purchase mode. Follow these steps carefully to ensure successful activation.

### Steps to Follow

1. Log in to the Submit01 Server

   - Open terminal or SSH client.
   - Connect to the server

2. Ensure you are logged in to Production

- AWS Logging

```bash
$ BROWSER=true aws configure sso

SSO start URL [None]: https://exasol-ct.awsapps.com/start#/
SSO Region [None]:  us-east-1
Attempting to automatically open the SSO authorization page in  your default browser.
If the browser does not open or you wish to use a different   device to authorize this request, open the following URL:
https://device.sso.us-east-1.amazonaws.com/
Then enter the code:
<code>
Using the account ID <prod-saas>
The only role available to you is: <AccessPolicy>
Using the role name "<AccessPolicy>"
CLI default client Region [None]:   eu-central-1
CLI default output format [None]:   Json
CLI profile name : <define the profile name>
To use this profile, specify the profile name using --profile,  as shown:
aws s3 ls --profile <profile_name>
```

3. Get the Ord ID (Account ID) from the customer (org id from the UI)

4. Activate subscription

```bash
aws lambda invoke --function-name prod-aws-activate-subscription --cli-binary-format raw-in-base64-out --profile <profile> --payload '{"orgId": "<ordId>", "amount": 100, "description": "Lorem Ipsum", "enddate": "2024-12-31"}' output.json
```

Note:

- In order to activate the subscription, you must provide the following information:
- Amount which is in Dollars (USD)
- Organization ID (Ord ID)
- Description: Description should have a meaning (add a ticket ID if needed)
- End Date: The end date of the subscription (it's an argument required just to activate the subscription, it will not expire even the date passed)

Final Step:

Once the lambda is invoked, check the console for the response. If the response if 400 please contact SaaS Team.
