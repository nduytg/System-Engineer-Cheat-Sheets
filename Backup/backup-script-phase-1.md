# Rotate Backup Script Phase 1

- **Shell:** /bin/bash
- **Author:** nduytg
- **Version:** 1.0
- **Date:** 22/11/17
- **Tested on:** CentOS7

## Backup with timestamp
## Simply backup with rsync to a remote host
usage()
{
	echo "Welcome to my backup script^^"
	echo "Please type in the arguments as follow:"
	echo "./backup source_path remote_path remote_user remote_ip"
	echo ""
}

TODAY="$(date +"%Y-%m-%e_%H-%M")"
LAST_DAY="$(date -d  "1 minutes ago" +"%Y-%m-%e_%H-%M")"
echo $TODAY
echo $LAST_DAY

echo "Date backup: " "$TODAY" >> backup_log.log

usage

if [ $# -lt 4 ] ; then
	echo "ERROR: Not enough arguments" >> backup_log.log
	echo "Exit now" >> backup_log.log
	exit
fi

root@192.168.31.131:/root/backup

SOURCE_PATH=$1
REMOTE_PATH=$2
REMOTE_USER=$3
REMOTE_IP=$4

### Incremental backup via ssh before using Rsync
ssh $REMOTE_USER@$REMOTE_IP /bin/bash << EOF
## LATEST_BACKUP="$(ls -1 $REMOTE_PATH | sort -r | head -n1)"
cp -al $REMOTE_PATH/$LATEST_BACKUP $REMOTE_PATH/$TODAY
EOF
### ----------In progress------------

### Rsync from the system into the latest backup
rsync 	-av \
		--rsync-path="mkdir -p "$REMOTE_PATH" && rsync" \
--link-dest="$REMOTE_PATH/$LAST_DAY" \
		--dry-run \
		--delete \
		-e ssh "$SOURCE_PATH" "$REMOTE_USER@$REMOTE_IP:$REMOTE_PATH/$TODAY" \
	&& echo "Date:" "$TODAY" "backup completed!" >> backup_log.log \
	|| echo "Date:" "$TODAY" "backup failed!" >> backup_log.log
echo "Backup completed!"
