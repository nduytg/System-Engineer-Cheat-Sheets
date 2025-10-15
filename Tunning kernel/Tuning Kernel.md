# Linux Kernel Tuning Cheat Sheet

- **Author:** nduytg
- **Version:** 1.2
- **Date:** 2017-11-09
- **Tested on:** CentOS 7

Tune networking, filesystem, and security parameters via `/etc/sysctl.conf`.

## Preparation

```bash
sysctl -a
sudo cp /etc/sysctl.conf /etc/sysctl_backup.conf
sudo vi /etc/sysctl.conf
```

## Networking

```conf
# Congestion control
net.ipv4.tcp_congestion_control = htcp
net.ipv4.tcp_timestamps = 1
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_sack = 1

# Socket buffers
net.ipv4.tcp_rmem = 8192 87380 16777216
net.ipv4.udp_rmem_min = 16384
net.core.rmem_default = 262144
net.core.rmem_max = 16777216
net.ipv4.tcp_wmem = 8192 65536 16777216
net.ipv4.udp_wmem_min = 16384
net.core.wmem_default = 262144
net.core.wmem_max = 16777216
net.core.somaxconn = 16384
net.core.netdev_max_backlog = 16384
net.core.dev_weight = 64

# Connection tracking
net.nf_conntrack_max = 100000
net.netfilter.nf_conntrack_tcp_timeout_established = 600

# Security hardening
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 262144
net.ipv4.tcp_syn_retries = 2
net.ipv4.tcp_synack_retries = 2
net.ipv4.ip_forward = 0
net.ipv4.conf.all.forwarding = 0
net.ipv4.conf.default.forwarding = 0
net.ipv6.conf.all.forwarding = 0
net.ipv6.conf.default.forwarding = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0
net.ipv6.conf.default.accept_source_route = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1

# Connection lifecycle
net.ipv4.tcp_fin_timeout = 7
net.ipv4.tcp_keepalive_time = 300
net.ipv4.tcp_keepalive_probes = 5
net.ipv4.tcp_keepalive_intvl = 15

# ICMP
net.ipv4.icmp_echo_ignore_all = 0
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.icmp_ignore_bogus_error_responses = 1

# Misc networking
net.ipv4.conf.all.proxy_arp = 0
net.ipv4.ip_local_port_range = 16384 65535
net.ipv4.tcp_rfc1337 = 1
```

## Filesystem and memory

```conf
fs.file-max = 300000
vm.swappiness = 10
vm.dirty_background_ratio = 5
vm.dirty_ratio = 10
vm.overcommit_memory = 0
vm.overcommit_ratio = 50
```

## Kernel hardening and IPv6

```conf
kernel.randomize_va_space = 2
net.ipv6.conf.all.autoconf = 0
net.ipv6.conf.all.accept_ra = 0
net.ipv6.conf.default.autoconf = 0
net.ipv6.conf.default.accept_ra = 0
```

## Apply changes

```bash
sudo sysctl -p
```
