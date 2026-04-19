# Process and File Descriptor Limits

- **Author:** System Engineer Collective
- **Version:** 2.0
- **Date:** 2024-05-01
- **Tested on:** Arch Linux, Debian 12, Ubuntu 22.04

Processes inherit two layers of ceilings: shell-oriented limits (soft and hard)
and the kernel-wide maximum number of descriptors the system can allocate. Keep
all three aligned so services scale predictably.

## Understand the limit types

* **Soft limit** – Active threshold enforced for a running shell or service.
  Users can raise it up to the matching hard limit.
* **Hard limit** – Upper bound a non-root user can request. Root (or a systemd
  unit with the appropriate capability) can raise it further.
* **Kernel ceiling** – Applies to the entire system. File descriptors use
  `fs.file-max` and sockets additionally honor networking parameters such as
  `net.core.somaxconn`.

## Check your current limits

Review the shell defaults before you launch long-running services.

```bash
ulimit -a
ulimit -Sn
ulimit -Hn
```

Inspect a specific process by reading its `/proc` metadata.

```bash
pidof sshd
cat /proc/"$(pidof sshd)"/limits
ls /proc/"$(pidof sshd)"/fd | wc -l
```

`prlimit` offers the same data with clearer output and can modify values on the
fly.

```bash
sudo prlimit --pid $(pidof nginx)
sudo prlimit --pid $(pidof nginx) --nofile=65535:65535
```

## Apply temporary adjustments

`ulimit` changes apply to the current shell session and any children it spawns.
Use it for ad-hoc testing or in wrapper scripts executed by systemd units.

```bash
ulimit -n 65535
```

Modify a running service without restarting by targeting its PID with
`prlimit`.

```bash
sudo prlimit --pid 1234 --nofile=65535:65535
```

## Persist limits across reboots

### PAM limits for logins and services

Persist user-based limits through `/etc/security/limits.d/*.conf` or the legacy
`/etc/security/limits.conf`. Group entries (prefixed with `@`) keep settings in
sync for system users such as web or database services.

```conf
@nginx   soft  nofile  65535
@nginx   hard  nofile  65535
```

Confirm `pam_limits.so` is enabled for the relevant PAM stack (login shells,
SSH, display managers). For SSH this means setting `UsePAM yes` in
`/etc/ssh/sshd_config`.

### Systemd units

Override systemd unit files instead of editing upstream service definitions.

```bash
sudo systemctl edit nginx
```

```ini
[Service]
LimitNOFILE=65535
```

Reload and restart the service.

```bash
sudo systemctl daemon-reload
sudo systemctl restart nginx
```

### Kernel-wide descriptor pool

Raise the system ceiling with `sysctl` so user limits have room to grow.

```bash
sysctl fs.file-max
sudo sysctl -w fs.file-max=500000
```

Make the change persistent by dropping a file under `/etc/sysctl.d/`.

```bash
sudo tee /etc/sysctl.d/80-fd.conf <<'CONF'
fs.file-max = 500000
CONF

sudo sysctl --system
```

Monitor overall usage with `cat /proc/sys/fs/file-nr`. The second field shows
how many descriptors are currently allocated, which helps you size the ceiling
for busy hosts.

```bash
cat /proc/sys/fs/file-nr
```

## Troubleshooting tips

* If limits do not take effect for SSH sessions, double-check that `sshd` was
  restarted after editing its configuration and that no conflicting files exist
  under `/etc/security/limits.d/`.
* For containers, remember that cgroup controllers may impose additional limits
  (for example, `LimitNOFILE` in the unit file that launches the container). Use
  `systemd-cgls` or `crun exec --pidfile` to inspect them.
* Applications linked against `libcap` may drop privileges after start-up. They
  must raise their soft limit before relinquishing `CAP_SYS_RESOURCE`.
