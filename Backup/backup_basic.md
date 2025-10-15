# Backup Script with Crontab

- **Author:** nduytg
- **Version:** 0.5
- **Date:** 21/11/17
- **Tested on:** CentOS7

#### Local Backup
yum install rsync
rsync –av --dry-run --delete /home/nduytg/source/ /home/nduytg/backup/

#### External Backup (via SSH)
yum install ssh rsync
rsync -av --dry-run --delete -e ssh /home/nduytg/source/ root@192.168.31.131:/root/backup

#### Set up Private Key => Auto Login

#### Schedule backup jobs with crontab
crontab -e

## Backup every 2 minutes
*/2 * * * * rsync -av --delete -e ssh /home/nduytg/source/ root@192.168.31.131:/root/backup > /dev/null
