# Kinetic Platform Community Installer

Self-contained installer for deploying the Kinetic Platform on a fresh Linux server (Ubuntu, Redhat)

## What's in the Box

| File | Purpose |
|---|---|
| `.config`| USERNAME and PASSWORD for Kinetic CLI |
| `kinetic`| Kinetic CLI |
| `kinetic-platform-installer-*.tar.gz`| Pre-packaged install files |
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
   cd kinetic-platform-community-edition
   tar xzf kinetic-platform-installer*.tar.gz
   ```

3. Run the installer as root:
   ```bash
   # Starter mode with custom domain and custom admin password 
   sudo ./kinetic-platform-installer/install.sh --starter --domain example.com --password adminpassword
   ```


You should see output like this: 


  ╔════════════════════════════════════════╗
  ║     Kinetic Platform Installer v2      ║
  ╚════════════════════════════════════════╝


=== Validating Prerequisites ===

[OK] Prerequisites validated

=== Starter Mode ===

[INFO] Domain:            example.com
[INFO] Ingress:           traefik
[INFO] Ports:             80/443 (hostPort)
[INFO] Elasticsearch:     in-cluster
[INFO] NFS Server IP:     192.168.86.55
[INFO] System Username:   admin

[OK] Demo defaults applied

...


4. Wait for installation to complete (~10-15 minutes). Credentials are printed at the end.


5. Meantime - edit hostsfile so you point example.com (or whatever name you used) to the IP address given
(Above sample shows 192.168.86.55)


(Optional Step 6) 
6. # Set the default email address to receive system level emails (disk full, server unhealthy)
# Usage: kinetic system smtp-set <host> <port> <tls> <username> <password> <from_name> <from_address> [validation_email]
./kinetic system smtp-set smtp.gmail.com 587 true joe.blow@gmail.com password Joe joe.blow@gmail.com youremail@domain.com


7. # Build your first tennant -- I call it "first"
# Usage: kinetic system space-create <name> <admin_user> <email> <display_name> <password>
./kinetic system space-create first joe youremail@domain.com Joe joe1

8. # Open your Kinetic Platform administrator console
Open: `https://example.com/app/console/`
Your browser will warn about a self-signed certificate — click through to proceed.
# License the system (License defaults to 1000 submissions - then need to restart core server)

9. # Open your Kinetic Platform first space administrator console
Open: `https://first.example.com/app/console/`

10. # Open your Kinetic Platform first space as a user 
https://first.example.com/







[[AI WORLD // incomplete directions]]
11. # DOWNLOAD MCP and Skills
https://github.com/kineticdata/kinetic-platform-mgnt-mcp-server
https://github.com/kineticdata/kinetic-platform-ai-skills


x. # Build App
(claude)
connect the mcp server to Kinetic Platform
load the Skills for Kinetic Platform

Build me a simple helpdesk application on the Kinetic Platform



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

