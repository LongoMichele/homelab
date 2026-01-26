# 🖥️ Homelab

This repository contains a simple homelab setup based on Docker Compose.

Services are split into multiple compose files under the `services` directory and managed through a custom `compose` script.  
The setup also includes a Restic-based backup system to handle periodic backups.

## 🚀 Requirements

1. Install [Docker](https://docs.docker.com/engine/install/) to run the services
2. Install [Restic](https://restic.net/#installation) to create and handle backups

## ⚡️ Docker

The `compose` script will run services in the `docker-compose.yml` file and all the ones in the `.yml` files under the `services` directory.\
To create a new container add it to an existing file or create a new `.yml` file.

### 🛠️ Commands

Here is the list of the commands for the compose script:

```bash
# Starts all containers and detach
sh compose detach

# Starts all containers
sh compose up

# Stops all servies and removes orphans
sh compose down

# Updates all container images
sh compose pull
```

### 🔁 Autorun & service

The compose script can be run as a systemd service creating a dedicated file in `/etc/systemd/system/`.\
The contents of the `homelab.service` should be similar to:

```ini
[Unit]
Description=Start homelab containers
After=docker.service network-online.target
Wants=docker.service

[Service]
Type=oneshot
WorkingDirectory=/path/to/homelab
ExecStart=/path/to/homelab/compose detach
RemainAfterExit=yes
# RequiresMountsFor=/mnt/drive # Include the mount point if needed

[Install]
WantedBy=multi-user.target
```

## 💾 Restic

The `backup` script will create a new backup and save all informations in a `.log` file.

Before running the script a new repository needs to be initialized with `restic init --repo /path/to/backups`.\
The password needs to be set in the `.env` file to be able to create new backups.

### 🔁 Autorun & service

For a better backing up system the script can be run as a systemd service creating a dedicated `restic.service` and `restic.timer`.\
The contents of `restic.service` and `restic.timer` should be similar to:

```ini restic.service
[Unit]
Description=Restic homelab backup
Wants=network-online.target
After=network-online.target docker.service

[Service]
Type=oneshot
WorkingDirectory=/path/to/homelab
ExecStart=/path/to/homelab/backup
StandardOutput=journal
StandardError=journal
# RequiresMountsFor=/mnt/drive # Include the mount point if needed

[Install]
WantedBy=multi-user.target
```

```ini restic.timer
[Unit]
Description=Restic homelab daily backup

[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

## 🔐 Environment variables

The `.env` file is used to store environment variables required by services and scripts.

It currently contains variables used in both bash scripts and docker services.\
Scripts and services require an `.env` file to run — use `.env.sample` as a starting point.

