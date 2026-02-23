---
tool_name: confd_client
doc_type: guide
category: gpu
technical_entities:
  - GPU
  - NVIDIA
  - persistence daemon
  - systemd
summary: >
  How to set up the NVIDIA persistence daemon on Exasol cluster hosts —
  prevents driver unloading, 3-step systemd configuration.
---

# Set Up the NVIDIA Persistence Daemon

The NVIDIA kernel driver must be loaded and running for applications to use the
GPU. Without the persistence daemon, the driver may unload when no application
is using the GPU, causing startup delays.

## Step 1: Create or Choose a User

Run the service as a normal user without sudo privileges. Use the
`nvidia-persistenced` user if it exists:

```bash
cat /etc/passwd | grep nvidia-persistenced
export PERSISTENCED_USER=nvidia-persistenced
```

## Step 2: Configure the Service

```bash
sudo mkdir -p /etc/systemd/system/nvidia-persistenced.service.d/
cat << EOF | sudo tee -a "/etc/systemd/system/nvidia-persistenced.service.d/override.conf"
[Service]
ExecStart=
ExecStart=/usr/bin/nvidia-persistenced --user ${PERSISTENCED_USER} --persistence-mode --verbose
EOF
```

## Step 3: Restart the Service

```bash
sudo systemctl daemon-reload
[[ $(sudo systemctl is-enabled nvidia-persistenced.service) == "disabled" ]] && sudo systemctl enable nvidia-persistenced.service
sudo systemctl restart nvidia-persistenced.service
```

> **Note:** The legacy persistence mode is less robust and approaching end of
> support. Use the persistence daemon instead.
