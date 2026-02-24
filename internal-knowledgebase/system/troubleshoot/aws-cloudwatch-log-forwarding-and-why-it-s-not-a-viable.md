---
tool_name: cos
doc_type: troubleshoot
category: system
title: "AWS Cloudwatch Log Forwarding and why it's not a viable monitoring solution"
summary: "The main objective of this project was to build a log-parsing monitoring solution. This article shows why this is not a viable, robust or reliable solution."
---
# AWS Cloudwatch Log Forwarding and why it's not a viable monitoring solution

## Brief Overview

The main objective of this project was to build a log-parsing monitoring solution. This article shows why this is not a viable, robust or reliable solution.

Installing the agent, generating the config for it, putting it to SSM and making everything work together was not very hard. After all, this is Cloudwatch, a solution that has been maintained and improved for many years. I have even made a python-based tool to install the agent, generate the SSM config, apply it and start forwarding logs/metrics. Github repos can be found in the URLs below:

<https://github.exasol.com/seyidagha-aslanov/cwagent-exasol> - First iteration, with metrics

<https://github.exasol.com/seyidagha-aslanov/cwlogger-exasol> - Second iteration, with logs

Below I will explain why log-based monitoring with Cloudwatch, doesn't work.

## Explanation

## Part I. The stuff that work

There are a lot of things that work with the solution that we planed. We can easily gather logs from our desired locations and forward them to AWS cloudwatch for storage and "analysis". The [cwlogger-exasol](https://github.exasol.com/seyidagha-aslanov/cwlogger-exasol) tool does pretty much everything for you. Download, unpack and run the tool with the --install parameter and it will:

* Install the Cloudwatch Agent on all the Exasol nodes
* Generate a config file based on the Jinja2 template under /package/ssm_templates/
* Upload the config file to SSM
* Apply the config on the Cloudwatch Agent

After this logs will start flowing to AWS Cloudwatch

![](images/exa-Seyidagha_0-1616058987712.png)*Image 1. AWS Cloudwatch Log Stream Created According to the Jinja2 template*

As seen from the image above, the logs are being forwarded as expcted. They're "tagged" with the node hostname and the log-stream name is also conveniently named after the cluster name. This is all nice and good, but what does this give us?

## Part II. The stuff that don't work

### Part II.a. The metric filter

AWS Cloudwatch has a feature called metric filters, which creates metrics based on the contents of the ingested logs. This works... sometimes. This a very unreliable way of generating metrics based on the contents of the logs. The biggest issue is that the metric filter captures logs sometimes, but sometimes it just doesn't. Since the metric is not reliable, it's practically useless.

### Part II.b. Alarming is very limited

You might say, who needs metrics based on logs? Just send out the important logs. Well, the AWS alarming system creates alarms based on metrics only. I believe metric filters are there for this reason only. Therefore, the workflow of the Alarming should be Log > Metric > Alarm. The conversion of logs to metrics, as said in the section above, is very unreliable, thus rendering alarming also unreliable and as said before, practically useless.

### Part II.c. Metrics provided by Cloudwatch are "you get, what you get"

If you want to use the metrics which can be provided by the native Cloudwatch agent you're limited to CPU, RAM, SWAP, HDD (only if there's a FS persistent). Since our databases' data nodes' FS is SDFS (proprietary) Cloudwatch doesn't consider monitoring it. Well, we can use the AWS-native metrics, the ones that are provided with every EC2 instance, right? Well, we can use CPU, NET and Instance Status metrics, but not HDD... well, at least, not always. Apparently, the default, disk monitoring metrics are for some high-level instances only. Collectd requires additional plugins to be installed, which beats the idea of this being "100% cloud-native".

![](images/exa-Seyidagha_0-1616060486929.png)*Image 2. CPU, Instance state, Net Metrics being provided, but no HDD metrics*## Part III. Conclusion

Overall, I believe this solution is not the way to go. It is unreliable, has too many failure points and blatantly doesn't work sometimes. I believe we should work on a cloud-agnostic monitoring tool, which provides information to the customer's cloud environment. In my opinion, the medium providing the information is not important, as long as the information reaches the destination in a usable and understandable manner.
