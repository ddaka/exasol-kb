---
tool_name: c4
doc_type: guide
category: c4 Deployments
title: "c4 Creating Deployments"
summary: "To deploy Exasol on AWS, use `c4 aws play`:"
---
# c4 Creating Deployments

## AWS Deployment

To deploy Exasol on AWS, use `c4 aws play`:

```bash
c4 aws play -i <config_file>
```

**Example**:
```bash
c4 aws play -i ~/exasol/prod_cluster.yaml
```

## Required Parameters for AWS

The following parameters must be set for AWS deployments (via config file or command line):

**Essential parameters**:
- `CCC_AWS_REGION` - AWS region (e.g., `us-east-1`)
- `CCC_AWS_ACCESS_KEY_ID` - AWS access key
- `CCC_AWS_SECRET_ACCESS_KEY` - AWS secret key
- `CCC_PLAY_DATABASE_NAME` - Database name
- `CCC_DB_PASSWORD` - Database password
- `CCC_AWS_INSTANCE_TYPE` - EC2 instance type
- `CCC_AWS_NODES` - Number of nodes

## Example Configuration File

**YAML format** (`prod_cluster.yaml`):
```yaml
aws:
  region: us-east-1
  access_key_id: YOUR_ACCESS_KEY
  secret_access_key: YOUR_SECRET_KEY
  instance_type: c5d.2xlarge
  nodes: 4

play:
  database_name: production_db

db:
  password: secure_password_here
```

## Deployment Process

1. **Prepare configuration file** with all required parameters
2. **Run c4 play command**:
   ```bash
   c4 aws play -i config.yaml
   ```
3. **Monitor deployment** using `c4 ps`
4. **Wait for all nodes** to reach stage "d" (database running)

## Monitor Deployment Progress

During deployment, you can monitor progress:

```bash
c4 ps
```

Or connect to see deployment logs:

```bash
c4 connect -i PLAY_ID
```

## Deployment Stages

Deployments progress through stages:

| Stage | Description |
|-------|-------------|
| **a** | Cloud resources being allocated |
| **b** | Host booting and running startup scripts |
| **c** | COS running and reachable |
| **d** | Database running and reachable |

**Deployment complete** when all database nodes at stage **d**.

## Best Practices

**Use configuration files**
- Version control your configurations
- Easier to review and replicate
- Reduces command-line errors

**Test in staging first**
- Verify configuration works
- Catch issues early

**Document your deployments**
- Save PLAY_ID for future reference
- Note deployment date and purpose

**Secure credentials**
- Never commit secrets to version control
- Use environment variables for sensitive data

## Troubleshooting

### Deployment Stuck at Stage "a"

**Possible causes**:
- AWS credentials invalid
- Insufficient AWS permissions
- Region unavailable

**Solution**:
- Verify AWS credentials
- Check AWS console for errors
- Review CloudFormation stack

### Deployment Failed

**Check**:
- `c4 ps` for error messages
- AWS CloudFormation console
- c4 deployment logs via `c4 connect`

## Related Documentation

- [c4 Overview](c4_overview.md)
- [c4 Configuration](c4_configuration.md)
- [c4 Monitoring Deployments](c4_monitoring.md)
- [Configuration Parameters Reference](https://docs.exasol.com/db/latest/administration/aws/c4/c4_parameters.htm)
- [Deploy Exasol on AWS](https://docs.exasol.com/db/latest/administration/aws/installation.htm)
