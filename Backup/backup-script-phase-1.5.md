# Rotate Backup Script Phase 1.5

- **Shell:** /bin/bash
- **Author:** nduytg
- **Version:** 1.3
- **Date:** 23/11/17
- **Tested on:** CentOS7

## Backup with timestamp
## Simply pulling backup with rsync FROM a remote host
usage()
{
	echo "Welcome to my backup script^^"
	echo "Please type in the arguments as follow:"
	echo "./backup SOURCE_PATH SOURCE_USER SOURCE_IP LOCAL_PATH"
	echo ""
}
./backup_script.sh /home/nduytg/source nduytg 192.168.31.130 /root/backup

echo "-----+++Backup Task+++-----"
TODAY="$(date +"%Y-%m-%e_%H-%M")"
echo "Date backup: " "$TODAY" >> backup_log.log

usage

if [ $# -lt 4 ] ; then
	echo "ERROR: Not enough arguments" >> backup_log.log
	echo "Exit now" >> backup_log.log
	exit
fi

root@192.168.31.131:/root/backup

SOURCE_PATH=$1
SOURCE_USER=$2
SOURCE_IP=$3
LOCAL_PATH=$4

### Step 1: Incremental Backup
LATEST_BACKUP="$(ls -1 $LOCAL_PATH | sort -r | head -n1)"
if [ "$LATEST_BACKUP" != "" ] && [ "$LATEST_BACKUP" != "$TODAY" ] ; then
	echo "Linking latest backup to save space" >> backup_log.log
	cp -al "$LOCAL_PATH/$LATEST_BACKUP" "$LOCAL_PATH/$TODAY" >> backup_log.log
else
	echo "No need to hardlink" >> backup_log.log
fi

### Step 2: Rsync from remote system into the latest backup
rsync 	-av \
		--dry-run \
		--delete \
		-e ssh "$SOURCE_USER@$SOURCE_IP:$SOURCE_PATH" "$LOCAL_PATH/$TODAY"\
	&& echo "Date:" "$TODAY" "backup completed!" >> backup_log.log \
	|| echo "Date:" "$TODAY" "backup failed!" >> backup_log.log
echo "Backup completed!"
