---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Monitoring Onboarding"
summary: "This is a step-by-step tutorial which guides you through onboarding a new cluster/customer."
---
# Monitoring Onboarding

## Overview

This is a step-by-step tutorial which guides you through onboarding a new cluster/customer.

## Prerequisites

1. Cluster-ID (unique identifier)
    - please ask Dren or Marcel for a CLUSTER-ID
        - some clusters and customers already have existing IDs
        - some clusters and customers need to be created
    - SaaS
        - use the clusterid from the deployment
2. Monitoring Region
    - EU
    - US
3. Customer Name aka clientid from Salesforce (Account Name)
4. Service Level from Salesforce (which Service Level does the cluster have)
    - Silver
    - Gold
    - Platinum
5. Exasol Monitoring User (used to connect to Exasol monitoring stack) --> done by Exasol
    - Name in Keeper: Monitoring User
    - Login Name + Naming convention: clusterXXXX --->> XXXX Cluster-ID (1-4 digits)
    - Password without symbols at least 20 characters long
    - this user and password is create before any scheduled meetings
    - the user and passoword is stored in Keeper
    - Dren or Marcel will use those in the backend of the monitoring stack to grant access
7. Database Monitoring User (used to collect database statistics) --> done by customer
    - Please refer to [Database User](https://docs.exasol.com/db/latest/planning/support.htm#Databaseaccess)
    - can be done together with the customer or by the customer themselves
8. The cluster can access hervester.exasol.com on all required ports
    - for details check [here](https://exasol.my.site.com/s/article/Exasol-Monitoring-Service-FAQ?language=en_US)
9. Cluster (and may be also the customer) are on-boarded (clientid and clusterid) and visible in https://grafana.harvester-int.exasol.com
    - data in the graphs and panels will only be shown if the agents are properly installed and connected to the monitoring stack
    - Dren or Marcel can enable new customers and clusters in Proust
10. The cluster-ID and the Exasol monitoring user (clusterXXXX) are granted access to Kafka
    - check with Dren or Marcel
11. Hardware monitoring
    - if hardware for Dell servers should be monitored ensure DELL OMSA is installed and running properly
12. Salesforce ticket for onboarding customer x and cluster y
    - only assign this ticket to Dren or Marcel if prerequisite steps 1 - 7 are fullfilled.

## How to &lt;Onboard cluster to monitoring&gt;

### Step 1

Ensure prerequisites are fullfilled. Most of the work is preparation! DO NOT MOVE ON to step 2 unless Step 1 is fullfilled.

### Step 2

CAUTION: Before running the installation, please create and fill in proper information in proust.config. You will need almost all information from the prerequisites.
- Install the agent as described here: (https://github.com/exasol/exasol-cloud-monitoring/tree/main/agent#install-agent)

### Step 3

- Set unlimited silence in Karma for customers that have Silver or Platinum - unless these customers have Incident Management
