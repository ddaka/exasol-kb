---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Install AWS CLI Session Manager plugin"
summary: "Install the AWS Session Manager plugin required for SSM-based access to SaaS environments."
---
# Install AWS CLI Session Manager plugin

## Purpose

SaaS access workflows use AWS SSM sessions instead of direct SSH. The Session Manager plugin is required for `aws ssm start-session`.

## Installation

Follow AWS official instructions for your OS:

- <https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html>

Homebrew example (macOS):

```shell
brew install --cask session-manager-plugin
```

## Validation

```shell
session-manager-plugin --version
```

If installation rights are restricted, contact ITS/SPOC.


