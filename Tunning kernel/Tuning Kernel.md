# Linux Kernel Tuning Essentials

- **Author:** System Engineer Collective
- **Version:** 2.0
- **Date:** 2024-05-01
- **Tested on:** Arch Linux, Debian 12, Ubuntu 22.04

These notes summarize the day-to-day kernel tuning tasks you will meet on most
Linux servers. They focus on practical inspection commands, temporary tweaks,
and the persistent configuration you need so the changes survive a reboot.

## Inspecting and applying kernel parameters

### Discover current settings

Use `sysctl` to list or query runtime kernel parameters. Pipe the full list
through `less` or `rg` when you are hunting for a specific key.

```bash
sudo sysctl -a | less
sudo sysctl net.core.somaxconn
cat /proc/sys/net/ipv4/tcp_fin_timeout
```

For quick auditing, combine `sysctl --values` with command substitution so you
only print the numeric results.

```bash
sudo sysctl --values net.ipv4.ip_local_port_range
```

### Apply runtime changes

`sysctl -w` (or the longer `sysctl key=value` form) updates a kernel value
immediately until the next reboot.

```bash
sudo sysctl -w net.core.somaxconn=32768
sudo sysctl vm.dirty_ratio=10
```

Alternatively, write straight to the `/proc/sys` interface when you are working
inside automation that already has root privileges.

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/tcp_timestamps
```

### Make changes persistent

Drop-in configuration files keep your tuning reproducible. Create a descriptive
file under `/etc/sysctl.d/` and reapply all settings with `sysctl --system`.

```bash
sudo tee /etc/sysctl.d/99-performance.conf <<'CONF'
net.core.somaxconn = 32768
vm.swappiness = 10
CONF

sudo sysctl --system
```

Most distributions also read `/etc/sysctl.conf`. Use whichever location best
fits your configuration management story, but keep related options grouped to
simplify reviews.

## User, process, and kernel limits

Per-process limits (for example, the number of open files) and global kernel
limits complement each other:

* **Soft limit** – The active ceiling enforced for a shell or process. Users
  can raise it up to the matching hard limit.
* **Hard limit** – The maximum a non-root user can request. Only root or
  privileged services can expand this boundary.
* **Kernel-wide limit** – A system ceiling that applies regardless of process
  ownership. For file descriptors this is `fs.file-max`.

Inspect the current state with the shell built-ins or by reading the process
metadata directly.

```bash
ulimit -a
ulimit -Sn
ulimit -Hn
cat /proc/"$PID"/limits
```

Use `prlimit` when you need to review or adjust limits for a running service
without restarting it.

```bash
sudo prlimit --pid "$PID"
sudo prlimit --pid "$PID" --nofile=65535:65535
```

Persist per-user limits in `/etc/security/limits.d/*.conf` (or the legacy
`/etc/security/limits.conf`). Make sure PAM sessions load `pam_limits.so`—it is
enabled by default on modern distributions and through SSH when `UsePAM yes` is
set in `sshd_config`.

```conf
@nginx   soft  nofile  65535
@nginx   hard  nofile  65535
```

Match those values with a kernel-wide ceiling via `sysctl`.

```bash
sudo sysctl -w fs.file-max=200000
```

More detail on per-process configuration is provided in
[`process-and-file-limits.md`](./process-and-file-limits.md).

## I/O scheduler selection

Rotational disks and solid-state media benefit from different schedulers. Query
all block devices and their current policy with `lsblk`.

```bash
lsblk -d -o NAME,ROTA,SCHED
```

* **SSD/NVMe** – Choose `none` (previously called `noop`) or `mq-deadline` to
  minimize latency.
* **SATA SSDs on legacy kernels** – `deadline` strikes a balance between
  throughput and fairness.
* **Spinning disks** – `bfq` and `kyber` (where available) focus on consistent
  throughput for sequential workloads.

Switch schedulers at runtime by writing to the queue attribute.

```bash
echo mq-deadline | sudo tee /sys/block/sda/queue/scheduler
echo none | sudo tee /sys/block/nvme0n1/queue/scheduler
```

Persist the choice with a udev rule so it reapplies when the device comes back
online.

```bash
sudo tee /etc/udev/rules.d/60-io-scheduler.rules <<'RULE'
ACTION=="add|change", KERNEL=="nvme[0-9]n[0-9]", ATTR{queue/scheduler}="none"
RULE

sudo udevadm control --reload
```

On systems that boot via GRUB, you can also append `scsi_mod.use_blk_mq=1` or a
specific `elevator=` option to the kernel command line for legacy drivers.

## Swap and virtual memory

Start by reviewing the active swap devices and virtual memory policy.

```bash
swapon --show
free -h
sysctl vm.swappiness vm.vfs_cache_pressure
```

Tune swap behavior with the `vm.swappiness`, `vm.min_free_kbytes`, and dirty
page ratios. Lower swappiness (for example 10) keeps the working set in RAM for
latency-sensitive applications; higher values (60–100) favor offloading idle
pages on memory-constrained hosts.

```bash
sudo sysctl -w vm.swappiness=10
sudo sysctl -w vm.dirty_background_ratio=5
sudo sysctl -w vm.dirty_ratio=15
```

Store long-term settings in the same `/etc/sysctl.d/` file you use for other
kernel tunables so they survive reboots.

```bash
sudo tee /etc/sysctl.d/99-memory.conf <<'CONF'
vm.swappiness = 10
vm.vfs_cache_pressure = 75
CONF
```

### When to disable swap

Disabling swap altogether can help real-time trading systems, performance test
beds, or high-throughput databases that suffer when the kernel reclaims pages.
You still need enough physical RAM to absorb spikes. Turn swap off temporarily
and comment the entry in `/etc/fstab` to make the change permanent.

```bash
sudo swapoff -a
sudo sed -i 's/^\/\(\S\+\s\+\S\+\s\+swap\s\)/#\1/' /etc/fstab
```

Consider using a small zram device instead of traditional swap on laptops and
micro instances. It gives you headroom while keeping I/O on fast compressed
memory.

## Further reading

* [Arch Linux – Improving performance](https://wiki.archlinux.org/title/Improving_performance)
* Distribution tuning guides (RHEL Performance Tuning, SUSE Performance Guide)
* Hardware vendor documentation for NVMe and RAID controllers
