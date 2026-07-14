# Kinetic Platform Community Edition Installer

Self-contained installer for deploying the Kinetic Platform on a fresh Linux server (Ubuntu, Redhat)

## Prerequisites

- **Server:** Modern Linux with root/sudo SSH access
- **Resources:** Minimum 8 CPU, 32 GB RAM, 100 GB disk recommended
- **Internet:** Required for downloading k0s, Helm, and container images

## Install

0. Login to Linux server

1. Clone and extract repository to your server:
   ```bash
   git clone https://github.com/jdsundberg/kinetic-platform-community-edition.git
   cd kinetic-platform-community-edition
   tar xzf kinetic-platform-installer*.tar.gz
   ```

2. Run the installer as root
   ```bash
   # Note: This process takes about 10-15 minutes
   # Starter mode with custom domain and custom admin password 
   sudo ./kinetic-platform-installer/install.sh --starter --domain example.com --password adminpassword
   ```

3. Edit /etc/hosts (on server and on clients, unless you used a real IP/Name)
   ```bash
   192.168.86.55 example.com # IP address from install script
   ```

4. Build your first tennant -- I call it "first"
   ```bash
   # Usage: kinetic system space-create <name> <admin_user> <email> <display_name> <password>
   ./kinetic system space-create first joe joe@sample.com Joe joe1
   ```


## Consoles:
   Your browser will warn about a self-signed certificate — click through to proceed.

   System Admin: `https://example.com/app/console/`

   Admin for (First space): `https://first.example.com/app/console/`

   User for (First space): `https://first.example.com/`



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
  - Elasticsearch (in-cluster)
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



## Optional Steps
   ```bash
   Set the default email address to receive system level emails (disk full, server unhealthy)
   # Usage: kinetic system smtp-set <host> <port> <tls> <username> <password> <from_name> <from_address> [validation_email]
   ./kinetic system smtp-set smtp.gmail.com 587 true joe.blow@gmail.com password Joe joe.blow@gmail.com youremail@domain.com
   ```


## License the system
   License defaults to 1,000 submissions, the core server will shutdown, use the admin console to restart the core server
   See support@kineticdata.com for license assistance



## Build Apps with AI

Connect Claude to your platform with the Kinetic MCP server and skills library, then build applications by describing what you want:

```
Build me a simple helpdesk application on the Kinetic Platform
```

See [README-AI.md](README-AI.md) for setup.

