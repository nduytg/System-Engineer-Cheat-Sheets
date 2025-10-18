# Rsync Backup Managed by Cron

- **Author:** nduytg
- **Version:** 0.5
- **Date:** 2017-11-21
- **Tested on:** CentOS 7

Automate incremental backups with `rsync` and `cron`. Start with a local copy,
expand to a remote target over SSH, and finish by restarting the cron service to
apply schedule changes.

## Local dry run

```bash
sudo yum install rsync
rsync -av --dry-run --delete /home/nduytg/source/ /home/nduytg/backup/
```

## Remote dry run over SSH

```bash
sudo yum install openssh-clients rsync
rsync -av --dry-run --delete -e ssh /home/nduytg/source/ \
  root@192.168.31.131:/root/backup
```

Generate SSH keys and copy the public key to the remote host to avoid
interactive password prompts when cron executes the job.

## Schedule the synchronization

```bash
sudo crontab -e
```

Example entry (run every two minutes during testing):

```cron
*/2 * * * * rsync -av --delete -e ssh /home/nduytg/source/ \
  root@192.168.31.131:/root/backup > /dev/null 2>&1
```

Restart the cron daemon after editing the schedule.

```bash
sudo systemctl restart crond
```
