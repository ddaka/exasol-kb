---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Accessing the SaaS Main Dashboard"
summary: "For SaaS we are using Datadog as our primary monitoring tool. The main dashboard gives a high level status display for the entire SaaS environment."
---
# Accessing the SaaS Main Dashboard

## Overview

For SaaS we are using Datadog as our primary monitoring tool. The main dashboard gives a high level status display for the entire SaaS environment.

## Prerequisites

Web browser, working network.

## How to load the main SaaS Dashboard

## Step 1

Go to the main Exasol Datadog page <https://exasol.datadoghq.eu/>

## Step 2

Click the Dashboards on the left side of the page and next click Dashboard List.

![](images/Screenshot-2021-11-10-at-14.26.08.png)

## Step 3

Select the SaaS PROD Dashboard.

![](images/Screenshot-2021-11-14-at-15.46.44.png)

## Step 4

The PROD dashboard will open, and will look like this:

![](images/exaAnthonyL_0-1636901415772.png)

## Step 5.

Validate that there are no Alert/Warn or No Data messages showing on the Dashboard. If there are Alert/Warn or No Data messages check for a Playbook procedure for that error.

## Additional Notes

Reviewing this dashboard should be the first step taken in troubleshooting an issue in SaaS. It's a complex dashboard, so be sure to review the information carefully to determine if there are issues in the environment.

## Additional References

Note that there are also customer dashboards for each customer environment. You can see those in the overall dashboard list in Step 2, listed by customer UUID.
