# Rotate Backup Script (Phase 1)

- **Author:** nduytg
- **Version:** 1.0
- **Date:** 2017-11-22
- **Tested on:** CentOS 7

Phase 1 introduces timestamped backups. Each run creates a new directory on the
remote host and synchronizes the local source using `rsync`.

## Usage

```bash
./backup source_path remote_path remote_user remote_ip
```

## Script listing

```bash
#!/bin/bash
set -euo pipefail

usage() {
  cat <<'USAGE'
Usage: ./backup source_path remote_path remote_user remote_ip
USAGE
}

TODAY="$(date +%Y-%m-%d_%H-%M)"
LAST_MINUTE="$(date -d '1 minute ago' +%Y-%m-%d_%H-%M)"
LOG_FILE="backup_log.log"

if [[ $# -lt 4 ]]; then
  usage
  {
    echo "ERROR: Not enough arguments";
    echo "Exit now";
  } >>"${LOG_FILE}"
  exit 1
fi

SOURCE_PATH=$1
REMOTE_PATH=$2
REMOTE_USER=$3
REMOTE_IP=$4

RSYNC_CMD=(
  rsync -av \
    --rsync-path="mkdir -p ${REMOTE_PATH} && rsync" \
    --delete \
    -e ssh "${SOURCE_PATH}" \
    "${REMOTE_USER}@${REMOTE_IP}:${REMOTE_PATH}/${TODAY}"
)

if "${RSYNC_CMD[@]}"; then
  echo "Date: ${TODAY} backup completed!" >>"${LOG_FILE}"
else
  echo "Date: ${TODAY} backup failed!" >>"${LOG_FILE}"
fi

echo "Backup completed!"
```

Use `--link-dest` to enable incremental copies once a previous snapshot exists.
Set `LAST_MINUTE` to the most recent directory name on the remote host before
adding the flag.
