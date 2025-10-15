# Configure iptables (CentOS 7)

- **Author:** nduytg
- **Version:** 1.1
- **Date:** 2017-11-14
- **Tested on:** CentOS 7

Lock down a host so that only SSH, HTTP, and HTTPS are permitted inbound, while
DNS and NTP remain available for outbound lookups.

## Install and enable iptables services

```bash
sudo yum install -y iptables-services
sudo systemctl stop firewalld
sudo systemctl disable firewalld
sudo systemctl enable --now iptables
sudo systemctl status iptables
```

## Inspect and reset rules

```bash
sudo iptables -L
sudo iptables -S
sudo iptables -F
```

## Configure inbound policy

```bash
# Allow loopback traffic
sudo iptables -A INPUT -i lo -j ACCEPT

# Permit established/related sessions and drop invalid packets
sudo iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate INVALID -j DROP

# Allow SSH, HTTP, and HTTPS
sudo iptables -A INPUT -i <interface> -d <server_ip> -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -i <interface> -d <server_ip> -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -i <interface> -d <server_ip> -p tcp --dport 443 -j ACCEPT

# Optional: allow specific source ranges
sudo iptables -A INPUT -i <interface> -m iprange --src-range 10.0.0.20-10.0.0.35 -j ACCEPT

# Allow NTP responses
sudo iptables -A INPUT -p udp --sport 123 -j ACCEPT

# Default drop policy
sudo iptables -P INPUT DROP
```

## Configure outbound policy

```bash
# Allow loopback traffic
sudo iptables -A OUTPUT -o lo -j ACCEPT

# Permit established sessions
sudo iptables -A OUTPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT

# Allow outbound SSH and web traffic
sudo iptables -A OUTPUT -o <interface> -p tcp --sport 22 -j ACCEPT
sudo iptables -A OUTPUT -o <interface> -p tcp --sport 80 -j ACCEPT
sudo iptables -A OUTPUT -o <interface> -p tcp --sport 443 -j ACCEPT
sudo iptables -A OUTPUT -o <interface> -p tcp --dport 80 -j ACCEPT
sudo iptables -A OUTPUT -o <interface> -p tcp --dport 443 -j ACCEPT

# Allow DNS and NTP queries
sudo iptables -A OUTPUT -p udp --dport 53 -j ACCEPT
sudo iptables -A OUTPUT -p udp --dport 123 -j ACCEPT

# Default drop policy
sudo iptables -P OUTPUT DROP
```

## Disable forwarding

```bash
sudo iptables -P FORWARD DROP
```

## Persist the configuration

```bash
sudo service iptables save
# or
sudo iptables-save | sudo tee /etc/sysconfig/iptables
```

Restore saved rules on boot with `iptables-restore` if required.
