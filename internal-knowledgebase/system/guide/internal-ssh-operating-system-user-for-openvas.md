---
tool_name: cos
doc_type: guide
category: system
title: "INTERNAL: SSH Operating System User for OpenVAS"
summary: "Create and harden a dedicated OS account for OpenVAS SSH access on COS nodes."
---
# INTERNAL: SSH Operating System User for OpenVAS

## Purpose

Create a restricted operating system user that can be used by OpenVAS for SSH-based checks.

## Prerequisites

- Root or equivalent administrative access.
- Public key provided by the scanning platform.
- Target username and group (examples below use placeholders).

## Procedure

1. Create a dedicated local user account:

```bash
cosexec -art useradd -m <USER_NAME>
```

2. Lock password authentication for that account:

```bash
cosexec -art usermod -L <USER_NAME>
```

3. Create `.ssh` directory and key file:

```bash
cosexec -art mkdir -p /home/<USER_NAME>/.ssh
cosexec -art sh -c ': > /home/<USER_NAME>/.ssh/authorized_keys'
```

4. Add the OpenVAS public key to `authorized_keys`:

```text
/home/<USER_NAME>/.ssh/authorized_keys
```

5. Apply secure ownership and permissions:

```bash
cosexec -art chown -R <USER_NAME>:<USER_GROUP> /home/<USER_NAME>
cosexec -art chmod 700 /home/<USER_NAME>/.ssh
cosexec -art chmod 600 /home/<USER_NAME>/.ssh/authorized_keys
```

## Validation

Run a key-based SSH login test from the scanning host and verify shell access behaves as expected for the restricted account.

## Notes

- Keep this account dedicated to scanning use cases.
- If stricter hardening is required, add command restrictions in `authorized_keys` options.
