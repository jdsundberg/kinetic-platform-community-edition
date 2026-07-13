# Kinetic Platform Community Installer

Self-contained installer for deploying the Kinetic Platform on a fresh Linux server (Ubuntu, Redhat)

## What's in the Box

| File | Purpose |
|---|---|
| `install.sh` | Server-side installer — sets up k0s, NFS, Helm, and deploys the platform |
| `env.values.yaml.tpl` | Helm values template (placeholders replaced by install.sh) |
| `kinetic-platform-*.tgz` | Pre-packaged Helm chart |
| `README.md` | This file |

## Prerequisites

- **Server:** Modern Linux with root/sudo SSH access
- **Resources:** Minimum 8 CPU, 32 GB RAM, 100 GB disk recommended
- **Internet:** Required for downloading k0s, Helm, and container images

## Install

0. Login to server

1. Clone repository to your server:
   ```bash
   git clone https://github.com/jdsundberg/kinetic-platform-community-edition.git
   ```

2. Extract the install:
   ```bash
   tar xzf kinetic-platform-installer.tar.gz
   ```

3. Run the installer as root:
   ```bash
   # Starter mode with custom domain and custom admin password 
   sudo ./kinetic-platform-installer/install.sh --starter --domain kinetic.example.com --password s3cr3t
   ```

4. Wait for installation to complete (~10-15 minutes). Credentials are printed at the end.
   # Reward yourself with a nice refreshment


Then open: `https://<domain>/app/console/`

Your browser will warn about a self-signed certificate — click through to proceed.

## What Gets Installed (Server)

- **k0s** — Single-node Kubernetes cluster
- **NFS server** — Shared storage at `/srv/nfs/kinetic`
- **Helm 3** — Kubernetes package manager
- **Kinetic Platform** including:
  - Cassandra (in-cluster, single node)
  - PostgreSQL (in-cluster)
  - RabbitMQ (single node)
  - Core, Agent, System Coordinator
  - System Console, Space Console, OAS Console
  - Indexer, Bundles
  - Elasticsearch
  - Loghub + Log Collector

All services run with **1 replica** and no HPA. Scale up manually after install if needed.

## Credentials

Printed at the end of `install.sh` and stored in `env.values.yaml`:
- **System Username** — `admin`
- **System Password** — randomly generated (unless passed in at install)
- **Encryption Key** — in `essentials.platform.encryption_key`

## Uninstall

```bash
# Remove the Helm release
sudo helm uninstall kinetic-platform -n kinetic

# Remove NFS export
sudo rm -rf /srv/nfs/kinetic
sudo sed -i '/srv\/nfs\/kinetic/d' /etc/exports
sudo exportfs -ra

# Stop and remove k0s
sudo k0s stop
sudo k0s reset
```

