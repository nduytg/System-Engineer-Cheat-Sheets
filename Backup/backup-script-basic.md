# Rotate Backup Script (Phase 1)

- **Author:** nduytg
- **Version:** 0.0
- **Date:** 2017-11-xx
- **Tested on:** CentOS 7

This starter script keeps three rolling daily backups by cloning the most recent
snapshot and synchronizing changes from a source directory. It is intended to be
run manually while iterating on the rotation logic.

## Usage

```bash
./backup source_path remote_user remote_ip target_path
```

The script expects passwordless SSH access to the remote target. Update the
`SOURCE_PATH`, `REMOTE_USER`, `REMOTE_IP`, and `TARGET_PATH` arguments to match
your environment.

## Script listing

```bash
#!/bin/bash
set -euo pipefail

usage() {
  cat <<'USAGE'
Welcome to the rotating backup script.
Usage: ./backup source_path remote_user remote_ip target_path
USAGE
}

TODAY="$(date +%Y-%m-%d)"
YESTERDAY="$(date -d '1 day ago' +%Y-%m-%d)"
LOG_FILE="backup_log.log"

if [[ $# -lt 4 ]]; then
  usage
  {
    echo "Not enough arguments";
    echo "Exit now";
  } >>"${LOG_FILE}"
  exit 1
fi

SOURCE_PATH=$1
REMOTE_USER=$2
REMOTE_IP=$3
TARGET_PATH=$4

mkdir -p "${TARGET_PATH}"

# Rotate the previous snapshots.
rm -rf "${TARGET_PATH}/daily_backup.3"
if [[ -d "${TARGET_PATH}/daily_backup.2" ]]; then
  mv "${TARGET_PATH}/daily_backup.2" "${TARGET_PATH}/daily_backup.3"
fi
if [[ -d "${TARGET_PATH}/daily_backup.1" ]]; then
  mv "${TARGET_PATH}/daily_backup.1" "${TARGET_PATH}/daily_backup.2"
fi

# Clone the last backup to seed today's run.
if [[ -d "${TARGET_PATH}/${YESTERDAY}" ]]; then
  cp -al "${TARGET_PATH}/${YESTERDAY}" "${TARGET_PATH}/${TODAY}" \
    && echo "Linked ${YESTERDAY} -> ${TODAY}" >>"${LOG_FILE}"
else
  mkdir -p "${TARGET_PATH}/${TODAY}"
fi

RSYNC_CMD=(
  rsync -av --delete -e ssh "${SOURCE_PATH}" \
    "${REMOTE_USER}@${REMOTE_IP}:${TARGET_PATH}/${TODAY}"
)

if "${RSYNC_CMD[@]}"; then
  echo "Date: ${TODAY} backup completed!" >>"${LOG_FILE}"
else
  echo "Date: ${TODAY} backup failed!" >>"${LOG_FILE}"
fi

echo "Backup completed" >>"${LOG_FILE}"
```

Adjust logging paths and retention counts to satisfy production requirements
before automating the process with cron.
