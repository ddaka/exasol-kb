---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "SaaS - UI Monitoring - Synthetic Check"
summary: "Here we explain the steps and the way how we monitor UI through synthetic check."
---
# SaaS - UI Monitoring - Synthetic Check

## Overview

Here we explain the steps and the way how we monitor UI through synthetic check.

## Prerequisites

* Access to Datadog Production Account
* SaaS Production AWS Account

## Explanation

In order to check whether the secure connection using SSL to SaaS UI - cloud.exasol.com - is up and available and response time is under certain threshold, we run synthetic tests every 15 minutes. The tests will be executed against cloud.exasol.com on port 443, and it has two rules:

* if certificate is valid and its expire date is longer than 30 days, it will pass. If the certificate is not valid or it is about to expire within next 30 days, monitor will alert.
* if response time is longer than 2 seconds, monitor will trigger the alarm.

## Procedure

### Step 1

Go to Datadog Production Account

### Step 2

Navigate to Dashboards in left navigation menu

### Step 3

Search for ***SaaS PROD Dashboard*** and open the dashboard

### Step 4

Under ***Web Services***, click on ***Warning / Alert***. It will open the Web Services monitors in new tab.

### Step 5

On the list of ***Web Services*** monitors, click on ***[Synthetics] Synch Test on PROD.***

### Step 6

Check for how long the monitor has been in ***Warning / Alert*** status.

### Step 7

Check the test results details by going to Test Results section filtering the results by Alert:

![](images/TestResultsSynth.png)

Right now, we don't have any Alert from recent executed tests, but in case there are, it will look similar to the following:

![](images/TestResultsSynthetic.png)

### Step 8

Click on the test result and check what was the reason for Alert, it should be either SSL Certificate issue or Response Time is taking more than 2 seconds. Test details look like the following:

![](images/TestResult.png)

In the Assertions drop-down section you can see which of the conditions are failing:

* If Certificate is invalid (false) or Certificate will expire in less than 30 days, then follow the How-To in [this article](saas-ssl-cert-time-expiry.md) to get more information about SSL Certification Time Expiry.
* If Response Time is longer than 2 seconds (2000ms), then please contact Production Engineering Team.

## Additional References

[Datadog Synthetic Tests](https://docs.datadoghq.com/synthetics/)

[SSL Cert Time Expiry](saas-ssl-cert-time-expiry.md)
