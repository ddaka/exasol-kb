---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Exasol v7 compliancy with Korean government guidelines"
summary: "South Korea recently had a security incident of SK Telecom mass data breach, which shook the country including an Exasol customer. The customer therefore took some security checks..."
---
# Exasol v7 compliancy with Korean government guidelines

## Question

1. What is the usage of SSH services that are listening on port number 40000+?

2. Are those ports necessary for Exasol system to run?

3. Can we change those port numbers?

4. Do those SSH services use of @/ISCSIADM_ABSTRACT_NAMESPACE entry?

5. What will we see if we run the following Linux command on the nodes?

   ```bash
   ss -0pb | grep -EB1 --color "$((0x7255))|$((0x5293))|$((0x39393939))"
   ```

6. Is there any Exasol specific packet filtering using BPF filter

## Answer

1. They are internal message passing channels.

2. Yes, they are necessary for Exasol system to run.

3. No.

4. Exasol codes do not use it. But it is used as part of the iscsi systemd socket in OS. That is required.

5. The output is empty, there is nothing.

6. No.

## Additional References

South Korea recently had a security incident of SK Telecom mass data breach, which shook the country including an Exasol customer. The customer therefore took some security checks based on their government guidelines against Exasol v7.1, and got several questions. This article addresses their concerns.
