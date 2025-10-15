# Rotate Backup Script (Phase 2)

- **Author:** nduytg
- **Version:** 1.3
- **Date:** 2017-11-23
- **Tested on:** CentOS 7

Phase 2 enforces retention. When the number of local backups exceeds the
configured threshold, the script archives and ships the oldest snapshot to a
remote destination before pruning it locally.

## Usage

```bash
./backup-script-phase-2.sh /root/backup root@192.168.171.130:/root/backup-san
```

## Script listing

```bash
#!/bin/bash
set -euo pipefail

usage() {
  cat <<'USAGE'
Usage: ./backup source_path remote_path
USAGE
}

if [[ $# -lt 2 ]]; then
  usage
  exit 1
fi

SOURCE_PATH=$1
REMOTE_PATH=$2
MAX_BACKUPS=${MAX_BACKUPS:-7}
LOG_FILE="rotating_log.log"

NUMBER_OF_BACKUPS=$(find "${SOURCE_PATH}" -mindepth 1 -maxdepth 1 -type d | wc -l)

echo "======= Processing =======" >>"${LOG_FILE}"
echo "Timestamp: $(date +%Y-%m-%d_%H-%M)" >>"${LOG_FILE}"
echo "Existing backups: ${NUMBER_OF_BACKUPS}" >>"${LOG_FILE}"
echo "Max backups: ${MAX_BACKUPS}" >>"${LOG_FILE}"

if (( NUMBER_OF_BACKUPS <= MAX_BACKUPS )); then
  echo "No rotation required" >>"${LOG_FILE}"
  exit 0
fi

while (( NUMBER_OF_BACKUPS > MAX_BACKUPS )); do
  OLDEST_BACKUP=$(ls -1 "${SOURCE_PATH}" | sort | head -n1)
  [[ -z "${OLDEST_BACKUP}" ]] && break

  ARCHIVE="${SOURCE_PATH}/${OLDEST_BACKUP}.tar.gz"
  tar -cvzf "${ARCHIVE}" -C "${SOURCE_PATH}" "${OLDEST_BACKUP}" >>"${LOG_FILE}"
  rm -rf "${SOURCE_PATH}/${OLDEST_BACKUP}"

  RSYNC_CMD=(
    rsync -av --remove-source-files -e ssh "${ARCHIVE}" "${REMOTE_PATH}/"
  )

  if "${RSYNC_CMD[@]}"; then
    echo "Archive ${ARCHIVE} transferred" >>"${LOG_FILE}"
  else
    echo "Archive ${ARCHIVE} transfer failed" >>"${LOG_FILE}"
  fi

  NUMBER_OF_BACKUPS=$((NUMBER_OF_BACKUPS - 1))
  echo "Backups remaining: ${NUMBER_OF_BACKUPS}" >>"${LOG_FILE}"
done

echo "Rotation complete" >>"${LOG_FILE}"
```

Remove `--dry-run` when ready for production and point `REMOTE_PATH` to durable
storage (for example, a SAN or object store gateway).
