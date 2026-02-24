---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "How to create an AWS Support Ticket"
summary: "In this article, we will show you how to create an AWS Support ticket if needed. You can find detailed version of this playbook, on AWS website..."
---
# How to create an AWS Support Ticket

## Overview

In this article, we will show you how to create an AWS Support ticket if needed. You can find detailed version of this playbook, on AWS website ([link](https://docs.aws.amazon.com/awssupport/latest/user/case-management.html)).

## Prerequisites

Access to SaaS Production AWS accounts.

### Step 1

* Sign in to the AWS Console

### Step 2

* In the upper-right corner, choose **Support**, and then choose **Support Center**.

### Step 3

* Choose to **Create a case**.

### Step 4

### There are three options

* **Account and billing support**
* **Service limit increase**
* **Technical support**

***Technical support** is the right option in case of incidents (production system is impaired or down due to AWS infrastructure issues).*

### Step 5

Complete the case with details as needed, as shown in the screenshot below.

![](images/exaEdis_0-1644869094950.png)

**Service** – If your question affects multiple services, choose the service that's most applicable.

**Category** – Choose the category that best fits your use case.

**Severity** – The business plan is applied to SaaS Production accounts hence we can choose **Production system impaired** (4-hour response) or **Production system down** (1-hour response).

`Note:`
`Based on your category choice, you might be prompted for more information.`

After you specify the case type and classification, you can specify the description and how you want to be contacted.

![](images/exaEdis_1-1644869805138.png)

**Subject** – Enter a title that briefly describes your issue.

**Description** – This is the most important information that you provide to AWS Support. For most service and category combinations, a prompt suggests information that's most helpful for the fastest resolution.

**Attachments** – Screenshots and other attachments (less than 5 MB each) can be helpful.

**Preferred contact language** – Currently, you can choose English or Japanese.

**Contact methods** – Choose a contact method. We have a Business Plan so you can choose **Chat,** **Phone or Web**. If you choose **Phone**, you're prompted for a callback number.

**Additional contacts** – Enter the email addresses of people to be notified when the status of the case changes. If you're signed in as an IAM user, include your email address. If you're signed in with your email address and password, you don't need to include your email address.

Choose to **Submit** when your information is complete and you're ready to create the case.

## Additional References

[AWS Documentation - create a support ticket](https://docs.aws.amazon.com/awssupport/latest/user/case-management.html)
