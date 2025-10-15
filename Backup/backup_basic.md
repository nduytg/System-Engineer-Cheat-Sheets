# Backup Script with Cron

- **Author:** nduytg
- **Version:** 0.5
- **Date:** 2017-11-21
- **Tested on:** CentOS 7

Use `rsync` to protect local data and push a copy to a remote server on a
schedule managed by `cron`.

## Local backup

Install `rsync` and run a dry-run to validate which files will be synchronized.

```bash
sudo yum install rsync
rsync -av --dry-run --delete /home/nduytg/source/ /home/nduytg/backup/
```

## Remote backup over SSH

Install the SSH client and `rsync`, then copy the data to a remote host. Running
with `--dry-run` ensures the command behaves as expected before removing the
flag for production use.

```bash
sudo yum install openssh-clients rsync
rsync -av --dry-run --delete -e ssh /home/nduytg/source/ \
  root@192.168.31.131:/root/backup
```

Set up SSH key-based authentication between the control node and the backup
server so that the cron job can run unattended.

## Schedule the backup

Edit the root user's crontab and create a frequent job while testing.

```bash
sudo crontab -e
```

Example entry to synchronize the remote backup every two minutes:

```cron
*/2 * * * * rsync -av --delete -e ssh /home/nduytg/source/ \
  root@192.168.31.131:/root/backup > /dev/null 2>&1
```

Adjust the interval to match production requirements once verification is
complete.
