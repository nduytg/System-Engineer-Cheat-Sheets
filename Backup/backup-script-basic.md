# Rotate Backup Script 1

- **Shell:** /bin/bash
- **Author:** nduytg
- **Version:** 0.0
- **Date:** xx/11/17
- **Tested on:** CentOS7

usage()
{
	echo "Welcome to my backup script^^"
	echo "Please type in the arguments as follow:"
	echo "./backup source_path remote_user remote_ip target_path"
	echo ""
}

TODAY="$(date +"%Y-%m-%d")"
LAST_DAY_BACKUP="date -d "1 day ago" +"%Y-%m-%d""
echo "Date backup: " $TODAY >> backup_log.log
DATE="$(date +"%Y-%m-%d")"
RSYNC=/usr/bin/rsync
SSH=/usr/bin/ssh

### Sample Comand
rsync -av --dry-run --delete -e ssh /home/nduytg/source/ root@192.168.31.131:/root/backup

usage

if [ $# -lt 4 ] ; then
	echo "Not enough arguments" >> backup_log.log
	echo "Exit now" >> backup_log.log
	exit
fi

echo "Test arguments variables"
echo "$0"
echo "$1"
echo "$2"
echo "$3"
echo "$4"
echo "$#"

SOURCE_PATH=$1
REMOTE_USER=$2
REMOTE_IP=$3
TARGET_PATH=$4

BACKUP_CYCLE=7
CURRENT_BACKUPS="ls -1 | wc -l"

### Check target path if it exists
if [ ! -d "$TARGET_PATH" ] ; then
	mkdir "$TARGET_PATH"
fi

#### Rotate backups
### STEP 1: Remove the oldest backup, if it is exists:
if [ -d "$TARGET_PATH/daily_backup.3" ] ; then
	rm -rf "$TARGET_PATH/daily_backup.3"
fi

### Step 2: Shift the middle snapshots(s) back one by one, if they exists
if [ -d "$TARGET_PATH/daily_backup.2" ] ; then
	mv "$TARGET_PATH/daily_backup.2" "$TARGET_PATH/daily_backup.3"
fi

if [ -d "$TARGET_PATH/daily_backup.1" ] ; then
	mv "$TARGET_PATH/daily_backup.1" "$TARGET_PATH/daily_backup.2"
fi

### Incremental Backup
cp -al $TARGET_PATH/$LAST_DAY_BACKUP $TARGET_PATH/$TODAY \
	&& echo Linking last backup complete >> backup_log.log \
	|| echo Failed copy source last backup >> backup_log.log

### Step 4: Rsync from the system into the latest backup
rsync 	-av \
		--dry-run \
		--delete \
		-e ssh "$SOURCE_PATH" "$REMOTE_USER@$REMOTE_IP:$TARGET_PATH/$TODAY" \
	&& echo "Date:" $TODAY "backup completed!" >> backup_log.log \
	|| echo "Date:" $TODAY "backup failed!" >> backup_log.log \


echo "Backup completed" >> backup_log.log
