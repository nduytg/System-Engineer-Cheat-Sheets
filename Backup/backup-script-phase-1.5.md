# Rotate Backup Script (Phase 1.5)

- **Author:** nduytg
- **Version:** 1.3
- **Date:** 2017-11-23
- **Tested on:** CentOS 7

Phase 1.5 reverses the data flow: pull backups from a remote system to local
storage. Hard links reduce disk usage by reusing unchanged files between runs.

## Usage

```bash
./backup SOURCE_PATH SOURCE_USER SOURCE_IP LOCAL_PATH
```

Example:

```bash
./backup_script.sh /home/nduytg/source nduytg 192.168.31.130 /root/backup
```

## Script listing

```bash
#!/bin/bash
set -euo pipefail

usage() {
  cat <<'USAGE'
Usage: ./backup SOURCE_PATH SOURCE_USER SOURCE_IP LOCAL_PATH
USAGE
}

TODAY="$(date +%Y-%m-%d_%H-%M)"
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
SOURCE_USER=$2
SOURCE_IP=$3
LOCAL_PATH=$4

mkdir -p "${LOCAL_PATH}"
LATEST_BACKUP="$(ls -1 "${LOCAL_PATH}" | sort -r | head -n1)"

if [[ -n "${LATEST_BACKUP}" && "${LATEST_BACKUP}" != "${TODAY}" ]]; then
  echo "Linking latest backup to save space" >>"${LOG_FILE}"
  cp -al "${LOCAL_PATH}/${LATEST_BACKUP}" "${LOCAL_PATH}/${TODAY}" >>"${LOG_FILE}"
else
  mkdir -p "${LOCAL_PATH}/${TODAY}"
fi

RSYNC_CMD=(
  rsync -av --delete -e ssh \
    "${SOURCE_USER}@${SOURCE_IP}:${SOURCE_PATH}" \
    "${LOCAL_PATH}/${TODAY}"
)

if "${RSYNC_CMD[@]}"; then
  echo "Date: ${TODAY} backup completed!" >>"${LOG_FILE}"
else
  echo "Date: ${TODAY} backup failed!" >>"${LOG_FILE}"
fi

echo "Backup completed!"
```

Remove `--dry-run` from the command array after testing to perform real
synchronizations.
