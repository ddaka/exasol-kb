---
tool_name: c4
doc_type: reference
category: c4 Timeouts
title: "c4 Timeout Configuration"
summary: "c4 uses configurable timeouts to prevent operations from hanging indefinitely. Use the `CCC_PLAY_TIMEOUT` environment variable to control timeout duration."
---
# c4 Timeout Configuration

## Overview

c4 uses configurable timeouts to prevent operations from hanging indefinitely. Use the `CCC_PLAY_TIMEOUT` environment variable to control timeout duration.

## Timeout Environment Variable

### CCC_PLAY_TIMEOUT

Sets the maximum duration for c4 operations before timeout.

**Format**: `<number><unit>`

**Units**:
- `s` = seconds
- `m` = minutes  
- `h` = hours

**Default**: Varies by operation (typically 10-30 minutes)

## Setting Timeouts

### Single Command

```bash
CCC_PLAY_TIMEOUT=30m c4 apply -P -i ./config
```

### Session-Wide

```bash
export CCC_PLAY_TIMEOUT=1h
c4 apply -P -i ./config
c4 update cluster -p PLAY_ID -t @exasol-2025.1.0
```

### Shell Configuration

Add to `~/.bashrc` or `~/.zshrc`:
```bash
export CCC_PLAY_TIMEOUT=45m
```

## Recommended Timeouts by Operation

| Operation | Recommended Timeout | Notes |
|-----------|---------------------|-------|
| **c4 ls** | 1m | List operations are fast |
| **c4 ps** | 2m | Deployment status check |
| **c4 apply** (small) | 30m | 3-node cluster |
| **c4 apply** (large) | 1h-2h | 10+ node cluster |
| **c4 update** | 50m-2h | Depends on cluster size |
| **c4 rm** | 30m | Removal operations |
| **c4 up/down** | 10m | Node start/stop |
| **c4 connect** | 5m | Connection establishment |

## Examples

### Quick Operations

```bash
# List deployments (1 minute timeout)
CCC_PLAY_TIMEOUT=1m c4 ls

# Check deployment status (2 minute timeout)
CCC_PLAY_TIMEOUT=2m c4 ps -p f7fdff8e
```

### Standard Operations

```bash
# Create 3-node deployment (30 minute timeout)
CCC_PLAY_TIMEOUT=30m c4 apply -P -i ./config

# Start nodes (10 minute timeout)
CCC_PLAY_TIMEOUT=10m c4 up f7fdff8e
```

### Long Operations

```bash
# Create large deployment (2 hour timeout)
CCC_PLAY_TIMEOUT=2h c4 apply -P -i ./config-large

# Update cluster (90 minute timeout)
CCC_PLAY_TIMEOUT=90m c4 update cluster -p f7fdff8e -t @exasol-2025.1.0
```

### Emergency Operations

```bash
# Extended timeout for slow networks (4 hour timeout)
CCC_PLAY_TIMEOUT=4h c4 apply -P -i ./config
```

## When to Increase Timeouts

### Network Issues

Slow or unstable network connections:
```bash
CCC_PLAY_TIMEOUT=2h c4 apply -P -i ./config
```

### Large Deployments

Clusters with many nodes:
```bash
# 10+ nodes
CCC_PLAY_TIMEOUT=90m c4 apply -P -i ./config-large
```

### AWS API Rate Limiting

When hitting AWS API limits:
```bash
CCC_PLAY_TIMEOUT=1h c4 apply -P -i ./config
```

### Complex Updates

Major version updates:
```bash
CCC_PLAY_TIMEOUT=2h c4 update cluster -p PLAY_ID -t @exasol-2025.1.0
```

### High AWS Load

During peak AWS usage:
```bash
CCC_PLAY_TIMEOUT=90m c4 apply -P -i ./config
```

## Timeout Errors

### Error Message

```
Error: Operation timed out after 30m
```

### Resolution

1. **Check operation status** in AWS console
2. **Increase timeout** and retry:
   ```bash
   CCC_PLAY_TIMEOUT=1h c4 apply -P -i ./config
   ```
3. **Monitor progress** with `c4 ps`

## Monitoring Long Operations

### Check Progress

While operation running:
```bash
# In another terminal
c4 ps -p PLAY_ID
```

### AWS Console Monitoring

Check AWS Console for:
- EC2 instance states
- CloudFormation stack events
- EBS volume attachment
- Network interface status

## Best Practices

**Set appropriate timeouts**
- Don't use very short timeouts
- Don't use unnecessarily long timeouts
- Match timeout to operation complexity

**Consider environment**
- Network speed
- AWS region performance
- Time of day (AWS load)

**Monitor long operations**
- Use `c4 ps` to check progress
- Watch AWS console
- Check logs for errors

**Document timeout choices**
- Record standard timeouts for your environment
- Share with team
- Update runbooks

**Test in non-production first**
- Determine realistic timeouts
- Account for variations
- Adjust based on experience

## Troubleshooting

### Operation Keeps Timing Out

**Check**:
1. Network connectivity
2. AWS service status
3. IAM permissions
4. Resource quotas

**Solutions**:
```bash
# Increase timeout significantly
CCC_PLAY_TIMEOUT=4h c4 apply -P -i ./config

# Check AWS status
curl https://status.aws.amazon.com/

# Verify credentials
aws sts get-caller-identity

# Check quotas
aws service-quotas list-service-quotas --service-code ec2
```

### Timeout After Long Wait

**Problem**: Operation times out but AWS resources created

**Solution**: Check AWS console
```bash
# List resources by tag
aws ec2 describe-instances --filters "Name=tag:play_id,Values=PLAY_ID"

# Clean up if needed
c4 rm -p PLAY_ID
```

### Unclear Timeout Needs

**Solution**: Start conservative
```bash
# Start with long timeout
CCC_PLAY_TIMEOUT=2h c4 apply -P -i ./config

# Note actual duration
# Adjust future timeouts based on experience
```

## Timeout Configuration File

### Create Timeout Wrapper Script

```bash
#!/bin/bash
# c4-wrapper.sh

# Set default timeout
export CCC_PLAY_TIMEOUT=45m

# Override for specific operations
case "$1" in
  apply)
    export CCC_PLAY_TIMEOUT=1h
    ;;
  update)
    export CCC_PLAY_TIMEOUT=90m
    ;;
  ls|ps)
    export CCC_PLAY_TIMEOUT=2m
    ;;
esac

# Execute c4 with all arguments
c4 "$@"
```

Usage:
```bash
chmod +x c4-wrapper.sh
./c4-wrapper.sh apply -P -i ./config
./c4-wrapper.sh ls
```

## Advanced: Per-Operation Timeout Logic

```bash
#!/bin/bash
# smart-timeout.sh

get_timeout() {
  local operation=$1
  local config_file=$2
  
  # Count nodes in config to estimate timeout
  if [[ "$operation" == "apply" && -f "$config_file" ]]; then
    local node_count=$(grep -c "node_size" "$config_file")
    if [[ $node_count -gt 10 ]]; then
      echo "2h"
    elif [[ $node_count -gt 5 ]]; then
      echo "1h"
    else
      echo "30m"
    fi
  else
    echo "30m"
  fi
}

# Usage
TIMEOUT=$(get_timeout "apply" "./config")
CCC_PLAY_TIMEOUT=$TIMEOUT c4 apply -P -i ./config
```

## Related Documentation

- [c4 Overview](c4_overview.md)
- [c4 Configuration](c4_configuration.md)
- [c4 Troubleshooting](c4_troubleshooting.md)
- [c4 Best Practices](c4_best_practices.md)
