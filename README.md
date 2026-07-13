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
- **Internet:** Required for downloading k0s, Helm, and container images
- **Resources:** Minimum 8 CPU, 32 GB RAM, 100 GB disk recommended

## Server Install

1. Copy the installer package to your server:
   ```bash
   scp kinetic-platform-installer.tar.gz user@server:~/
   ```

2. SSH into the server and extract:
   ```bash
   ssh user@server
   tar xzf kinetic-platform-installer.tar.gz
   cd kinetic-platform-installer
   ```

3. Run the installer as root:
   ```bash
   # Starter mode with custom domain and custom admin password 
   sudo ./install.sh --starter --domain kinetic.example.com --password s3cr3t
   ```

4. Wait for installation to complete (~10-15 minutes). Credentials are printed at the end.

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

## Ports

| Provider | HTTP NodePort | HTTPS NodePort |
|---|---|---|
| traefik | 31080 | 31443 |
| nginx | 30080 | 30443 |

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

## Troubleshooting

**Pods stuck in Pending/CrashLoopBackOff:**
```bash
sudo k0s kubectl get pods -n kinetic
sudo k0s kubectl describe pod <pod-name> -n kinetic
sudo k0s kubectl logs <pod-name> -n kinetic
```

**NFS mount failures:**
- Ensure NFS server is running: `systemctl status nfs-kernel-server`
- Check exports: `exportfs -v`
- Verify node IP matches `nfsClusterServer` in `env.values.yaml`

**Certificate issues:**
- Certs are in the `certs/` directory created by install.sh
- The platform-certificate-init job may skip if an `ingress` secret already exists
- Delete and reinstall if certs need to change: `kubectl delete secret ingress -n kinetic`

**CoreDNS not resolving the domain inside the cluster:**
- Verify configmap is Helm-managed: `kubectl get configmap coredns -n kube-system -o yaml`
- Restart CoreDNS: `kubectl rollout restart deployment coredns -n kube-system`

**504 Gateway Timeout (traefik):**
- Network policies may be blocking traffic — check: `kubectl get networkpolicy -n kinetic`
- Delete all policies if needed: `kubectl delete networkpolicy --all -n kinetic`

**local-access.sh — ports in use:**
- The script offers to use alternate ports 8080/8443
- Or kill the blocking process manually: `lsof -ti :443 | xargs kill`
