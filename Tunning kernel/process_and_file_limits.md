# Process and File Descriptor Limits

- **Author:** System Engineer Collective
- **Version:** 2.1
- **Date:** 2024-05-22
- **Tested on:** Arch Linux, Debian 12, Ubuntu 22.04

Processes inherit two layers of ceilings: shell-oriented limits (soft and hard)
and the kernel-wide maximum number of descriptors the system can allocate. Keep
all three aligned so services scale predictably.

## Understand the limit types

* **Soft limit** – The active threshold enforced for a running shell or service.
  Users can raise it up to the matching hard limit with `ulimit` or `prlimit`.
* **Hard limit** – The upper bound a non-root user can request. Root (or a
  systemd unit with `CAP_SYS_RESOURCE`) can raise it further.
* **Kernel ceilings** – Apply to the entire system. File descriptors use
  `fs.file-max` for the global pool, while `fs.nr_open` defines the highest value
  any single process can set as its hard limit. Networking workloads may also hit
  `net.core.somaxconn` or `net.ipv4.ip_local_port_range`.

`ulimit` is a shell built-in that reads or writes the soft (`-S`) and hard (`-H`)
limits backed by the kernel `RLIMIT_*` interface. Scripts and services that spawn
child processes inherit whatever values the parent exported.

## Check your current limits

Review the shell defaults before you launch long-running services.

```bash
ulimit -a
ulimit -Sn
ulimit -Hn
```

Map the shell settings to the actual kernel values and current consumption by
querying `/proc`.

```bash
pid=$(pidof sshd)
cat /proc/$pid/limits | grep -i "max open"
ls /proc/$pid/fd | wc -l
cat /proc/sys/fs/file-nr
cat /proc/sys/fs/nr_open
```

`prlimit` offers the same data with clearer output and can modify values on the
fly. Pair it with `--resource` for fine-grained inspection.

```bash
sudo prlimit --pid $pid --resource=NOFILE
sudo prlimit --pid $pid --nofile=65535:65535
```

For systemd-managed services, `systemctl show` exposes what the manager applied
at runtime.

```bash
systemctl show nginx -p LimitNOFILE,LimitNPROC
```

Containers introduce another layer: cgroup controllers may cap descriptors via
`LimitNOFILE`, `TasksMax`, or OCI runtime annotations. Use `systemd-cgls` or the
runtime tooling (`podman inspect`, `docker inspect`) to verify no additional
limits interfere with your tuning.

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
LimitCORE=infinity
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
sudo sysctl -w fs.nr_open=1048576
```

Make the change persistent by dropping a file under `/etc/sysctl.d/`.

```bash
sudo tee /etc/sysctl.d/80-fd.conf <<'CONF'
fs.file-max = 500000
fs.nr_open = 1048576
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
  `systemd-cgls` or runtime-specific inspectors to confirm.
* Applications linked against `libcap` may drop privileges after start-up. They
  must raise their soft limit before relinquishing `CAP_SYS_RESOURCE`.
* Track per-process usage during incidents with `sudo lsof -p $pid | wc -l` or
  `sudo ss -tanp` for sockets so you can justify permanent limit increases.
