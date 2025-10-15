# Rotate Backup Script Phase 2

- **Shell:** /bin/bash
- **Author:** nduytg
- **Version:** 1.3
- **Date:** 23/11/17
- **Tested on:** CentOS7

- Usage:
./backup-script-phase-2.sh /root/backup root@192.168.171.130:/root/backup-san

## Backup with timestamp
## Simply backup with rsync to a remote host
usage()
{
	echo "Welcome to my backup script^^"
	echo "Please type in the arguments as follow:"
	echo "./backup source_path remote_path"
	echo ""
}

SOURCE_PATH=$1
REMOTE_PATH=$2
TODAY="Time: $(date +"%Y-%m-%e_%H-%M")"
declare -i MAX_BACKUPS=7

echo "$TODAY"
LAST_DAY_BACKUP="date -d "1 day ago" +"%Y-%m-%d""
echo "=======+++Processing+++=======" >> rotating_log.log
echo "Timestamp: " "$TODAY" >> rotating_log.log

### Sample Comand
usage

if [ $# -lt 2 ] ; then
	echo "ERROR: Not enough arguments" >> rotating_log.log
	echo "Exit now" >> rotating_log.log
	exit
fi

### STEP 0: Count number of backups
NUMBER_OF_BACKUPs=$(ls -1 $SOURCE_PATH 2>/dev/null | wc -l)
echo "NUMBER_OF_BACKUPs:" "$NUMBER_OF_BACKUPs" >> rotating_log.log
echo "MAX_BACKUPS:" "$MAX_BACKUPS" >> rotating_log.log

### STEP 1: Check if need to rotating backup
if [ $NUMBER_OF_BACKUPs -le $MAX_BACKUPS ];
then
	echo "No need to rotating backup" >> rotating_log.log
	exit 0
else
	echo "Proceed to rotating backup" >> rotating_log.log
fi

## Check if number of backup > maximum backups
while [ $NUMBER_OF_BACKUPs -gt $MAX_BACKUPS ]
do
	echo "Archive old backups" >> rotating_log.log
	OLDEST_BACKUP="$(ls -1 $SOURCE_PATH | sort | head -n1)"
	echo "OLDEST_BACKUP: " "$OLDEST_BACKUP" >> rotating_log.log
	tar -cvzf "$SOURCE_PATH/$OLDEST_BACKUP.tar.gz" "$SOURCE_PATH/$OLDEST_BACKUP/"
	## Avoid delete the main backup folder
	if [ -n "$OLDEST_BACKUP" ]; then
		rm -rf "${SOURCE_PATH:?}/$OLDEST_BACKUP"
	fi

	### Step 2: Rsync to SAN
	rsync 	-av \
		--dry-run \
		--remove-source-files \
		-e ssh "$SOURCE_PATH/$OLDEST_BACKUP.tar.gz" "$REMOTE_PATH/" \
	&& echo "Timestamp: $(date +"%Y-%m-%e_%H-%M")" "archive" "$SOURCE_PATH/$OLDEST_BACKUP.tar.gz" "backup completed!" >> rotating_log.log \
	|| echo "Timestamp: $(date +"%Y-%m-%e_%H-%M")" "archive" "$SOURCE_PATH/$OLDEST_BACKUP.tar.gz" "backup failed!" >> rotating_log.log
	let "NUMBER_OF_BACKUPs--"
	echo "Number of backups: " "$NUMBER_OF_BACKUPs" >> rotating_log.log
done

echo "Rotating backup completed!" >> rotating_log.log
