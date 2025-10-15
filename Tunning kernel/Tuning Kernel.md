# Tunning Linux Kernel

- **Author:** nduytg
- **Version:** 1.2
- **Date:** 9/11/17
- **Tested on:** CentOS7

## Reference
https://klaver.it/linux/sysctl.conf
https://wiki.archlinux.org/index.php/Sysctl
https://www.kernel.org/doc/Documentation/sysctl/
https://www.kernel.org/doc/Documentation/networking/ip-sysctl.txt
https://www.speedguide.net/articles/linux-tweaking-121
https://vincent.bernat.im/en/blog/2014-tcp-time-wait-state-linux

### Show all parameters of Linux kernel
sysctl -a

##### Backup sysctl.conf before changing anything
sysctl -a > /etc/sysctl_backup.conf

##### Edit /etc/sysctl.conf to modify kernel parameters
vi /etc/sysctl.conf

###### Improving Network Performance

##### Congestion control
## Congestion control protocol can be changed between cubic and htcp (Hamilton TCP)
## Enable timestamps, this may cause overhead because it adds 10 bytes to each packets
## Enable window scaling (default = 1)
net.ipv4.tcp_congestion_control = htcp
net.ipv4.tcp_timestamps = 1
net.ipv4.tcp_window_scaling = 1

## Enable TCP SACK, which allow client to resend only lost packets, not all of them
## However, in some cases, TCP may consume more resources (CPU, RAM) and decrease network performance
net.ipv4.tcp_sack = 1


##### Increase socket buffer
## Increase Read Memory Buffer
## TCP Read Memory: Min - Default - Max
net.ipv4.tcp_rmem = 8192 87380 16777216
net.ipv4.udp_rmem_min = 16384
## Default read memory buffer of all receiving sockets (except TCP and UDP)
net.core.rmem_default = 262144
net.core.rmem_max = 16777216

## Increase Write Memory Buffer
## TCP Write Memory: Min - Default - Max
net.ipv4.tcp_wmem = 8192 65536 16777216
net.ipv4.udp_wmem_min = 16384
## Default read memory buffer of all sending sockets (except TCP and UDP)
net.core.wmem_default = 262144
net.core.wmem_max = 16777216

## Increase connection queue
net.core.somaxconn = 16384

## Improve packet processing queue, speed
net.core.netdev_max_backlog = 16384
net.core.dev_weight = 64

### Improve connection tracking
## For high-loaded servers
net.nf_conntrack_max = 100000
or
net.netfilter.nf_conntrack_max = 100000

## Decrease connection timeout in netfilter table
net.netfilter.nf_conntrack_tcp_timeout_established = 600

### Improving Network Security
## Prevent SYN Attack
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 262144
net.ipv4.tcp_syn_retries = 2
net.ipv4.tcp_synack_retries = 2

## Disable packet forwarding
net.ipv4.ip_forward = 0
net.ipv4.conf.all.forwarding = 0
net.ipv4.conf.default.forwarding = 0
net.ipv6.conf.all.forwarding = 0
net.ipv6.conf.default.forwarding = 0

## Disable IP source routing
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0
net.ipv6.conf.default.accept_source_route = 0

## Block ICMP redirect packets to prevent MITM attacks
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0

##### Prevent IP spoofing
## Enable reverse path filter to verify IPs
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1

### Decrease TCP FIN timeout
net.ipv4.tcp_fin_timeout = 7

### Decrease keep alive waiting time
net.ipv4.tcp_keepalive_time = 300
net.ipv4.tcp_keepalive_probes = 5
net.ipv4.tcp_keepalive_intvl = 15

##### Configure ICMP
net.ipv4.icmp_echo_ignore_all = 0
## Avoid smurf attack
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.icmp_ignore_bogus_error_responses = 1

## Disable Proxy ARP
net.ipv4.conf.all.proxy_arp = 0

## Configure local port range (only if server had a lot outbound connections)
net.ipv4.ip_local_port_range = 16384 65535

## Protect against TIME WAIT ASSASSINATION followed up RFC 1337
net.ipv4.tcp_rfc1337 = 1


###### Filesystem Tuning
### Increase open file limit
## For web/database/log servers which need a lot of open files
fs.file-max = 300000

###### Memory Tuning
### Decrease swapping
vm.swappiness = 10
vm.dirty_background_ratio = 5
vm.dirty_ratio = 10
vm.overcommit_memory = 0
vm.overcommit_ratio = 50

###### Kernel Hardening
## Prevent buffer/stack/heap exploits
## In CentOS7/RHEL7 exec-shield has been enabled by default and removed from /proc
## Use this option in CentOS 6 or other distro
kernel.exec-shield = 1
kernel.randomize_va_space = 2
kernel.pid_max = 4194303

###### IPv6
## Disable IPv6 by default
net.ipv6.conf.all.autoconf=0
net.ipv6.conf.all.accept_ra=0
net.ipv6.conf.default.autoconf=0
net.ipv6.conf.default.accept_ra=0
---

#### Reload changes
sysctl -p
