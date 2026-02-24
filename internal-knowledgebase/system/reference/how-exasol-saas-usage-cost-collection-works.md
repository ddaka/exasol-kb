---
tool_name: confd_client
doc_type: reference
category: system
title: "How Exasol SaaS Usage Cost Collection Works"
summary: "In this article, we explain how Exasol SaaS Usage Cost Collection Works."
---
# How Exasol SaaS Usage Cost Collection Works

## Overview

In this article, we explain how Exasol SaaS Usage Cost Collection Works.

## Explanation

Cost collection for Exasol SaaS can be enabled while deploying the SaaS environment or later by using `c4`.

There are 4 (four) c4 config parameters required for enabling cost collection:

* CCC_SAAS_AWS_SNS_USAGE_TOPIC
* CCC_SAAS_AWS_SNS_USAGE_TOPIC_NAME
* CCC_SAAS_AWS_USAGE_S3_BUCKET_NAME
* CCC_SAAS_AWS_USAGE_DATA_COLLECTION

`CCC_SAAS_AWS_SNS_USAGE_TOPIC:` Create AWS SNS Topic for Usage info collection. Must be set as `true` if SNS Topic creation is required for the initial SaaS deployment. If there is already an SNS topic it should be set as `false` and the existing SNS topic name should be provided by setting the c4 config parameter Amazon S3 event notification configurations don't allow overlapping suffixes in two (or more) rules if the prefixes are overlapping for the same event type. In this case, only one SNS Topic should be created per account.

`CCC_SAAS_AWS_SNS_USAGE_TOPIC_NAME:` AWS SNS Topic Name for Usage info collection. Must be set if Usage SNS Topic is required for the initial SaaS deployment. If there is already an SNS topic the value of this parameter should be the existing SNS topic name.

`CCC_SAAS_AWS_USAGE_S3_BUCKET_NAME:` AWS S3 bucket name where cost data is stored. Must be set if AWS Cost Collection is required for the initial SaaS deployment.

`CCC_SAAS_AWS_USAGE_DATA_COLLECTION:` Must be set true if AWS Usage collection is required for the initial SaaS deployment.

**How does it work?**

Based on provided config parameter values c4 will create the required resources in AWS. If there is no SNS topic exists and `CCC_SAAS_AWS_SNS_USAGE_TOPIC` set as `true` c4 will create a new SNS Topic and will add it as an event notification destination to the AWS S3 bucket.

The AWS S3 bucket name should be provided by `CCC_SAAS_AWS_USAGE_S3_BUCKET_NAME` config parameter.

![](images/exaFagani_2-1637749420991.png)

![](images/exaFagani_0-1637749346951.png)

c4 will also create 2 SQS Queues in the AWS SaaS Management account. These queues will be used by lambda function for calling the confd jobs named `saas_cost_collect` and `saas_cost_collect_internal_aggregation.` One of these queues will be used as a `dead-letter queue` in case if something went wrong with executing confd jobs.

![](images/exaFagani_1-1637749378018.png)

The lambda functions for creating subscriptions and executing confd jobs were also created by c4.

![](images/exaFagani_0-1637749526084.png)
