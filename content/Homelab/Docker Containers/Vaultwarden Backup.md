---
title: Vaultwarden Backup
draft: false
tags:
  - homelab
  - linux
  - ubuntu
  - docker
  - docker compose
  - vaultwarden
  - vaultwarden backup
  - rclone
  - google drive
  - omoikane
---

# Instructions

## 1. Configure rclone
```shell
docker run --rm -it \
  --mount type=volume,source=vaultwarden-rclone-data,target=/config/ \
  ttionya/vaultwarden-backup:latest \
  rclone config
```

For Google Drive you can use rclone's own client ID (which is rate-limited), or you can create your own client ID ([Google drive](https://rclone.org/drive/#making-your-own-client-id)).

## 2. Docker Compose

This assumes you use `BitwardenBackup` as remote name, otherwise change the `RCLONE_REMOTE_NAME` environment variable to whatever you used. This also assumes you use a volume for the vaultwarden container.

```yaml title="containers/vaultwarden/docker-compose.yml"
---
services:
  ...
  
  backup:
    image: ttionya/vaultwarden-backup:latest
    restart: always
    environment:
      RCLONE_REMOTE_NAME=BitwardenBackup
      RCLONE_REMOTE_DIR=/BitwardenBackup/
      CRON=0 3 * * *
      ZIP_ENABLE=true
      ZIP_PASSWORD=<password>
      BACKUP_KEEP_DAYS: 14
      PING_URL=<healthchecks.io link>
      TIMEZONE=Europe/Brussels
    volumes:
      - vaultwarden-data:/bitwarden/data/
      - vaultwarden-rclone-data:/config/

volumes:
  vaultwarden-data:
    name: vaultwarden-data
    # external: true # Only necessary if used in different compose file than vaultwarden.
  vaultwarden-rclone-data:
    external: true
    name: vaultwarden-rclone-data
```

> Sources:
> [GitHub - ttionya/vaultwarden-backup: Backup vaultwarden (formerly known as bitwarden\_rs) SQLite3/PostgreSQL/MySQL/MariaDB database by rclone. (Docker)](https://github.com/ttionya/vaultwarden-backup)
> [GitHub - rclone/rclone: "rsync for cloud storage" - Google Drive, S3, Dropbox, Backblaze B2, One Drive, Swift, Hubic, Wasabi, Google Cloud Storage, Azure Blob, Azure Files, Yandex Files](https://github.com/rclone/rclone)