# IPtables basic

- **Tested on:** CentOS7

              Configure IPtables
                Author: nduytg
          Version 1.1 - Date: 14/11/17



## Reference

## Configure IPtables to block all port except:
## Inbound TCP 22, TCP 80, TCP 443
- Outbound: UDP 53, UDP 123

#### Install IPtables
yum install iptables-services

#### Stop FirewallD Service and Start IPtables Service
systemctl stop firewalld
systemctl disable firewalld
systemctl start iptables
systemctl enable iptables
systemctl status iptables

#### List the currently configured iptables rules
iptables -L
iptables -S

#### Flush all current rules from iptables
iptables -F

#### Allow INBOUND connections

### Rules are evaluated in order, put busiet rules at the front!!
### Accept all traffic to the looback interface,
### which is necessary for many applications and services
iptables -A INPUT -i lo -j ACCEPT

#### Stateful table
### Allow traffic from existing connections or new connection related to these connections
iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT

### Block invalid packets
iptables -A INPUT -m conntrack --ctstate INVALID -j DROP

### Allow inbound port 22, 80, 443
iptables -A INPUT -i <input_interface> -d <server_IP> -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -i <input_interface> -d <server_IP> -p tcp --dport 443 -j ACCEPT
iptables -A INPUT -i <input_interface> -d <server_IP> -p tcp --dport 22 -j ACCEPT

### Allow IP range
iptables -A INPUT -i <input_interface> -m iprange --src-range 10.0.0.20-10.0.0.35 -j ACCEPT

### Block dropped packets
iptables -A INPUT -j LOG --log-prefix "IPTables-Dropped: "

### Allow DNS server (UDP/TCP) return result
### If use stateless table, enable the two below
iptables -A INPUT -i <input_interface> -p tcp --sport 53 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -i <input_interface> -p udp --sport 53 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT

### Allow NTP Server return result
iptables -A INPUT -p udp --sport 123 -j ACCEPT

### Except the listed above, other connections will be dropped
iptables -t filter -P INPUT DROP
---

#### Allow OUTBOUND connections
### Accept all traffic to the looback interface,
### which is necessary for many applications and services
iptables -A OUTPUT -o lo -j ACCEPT

### Allow Established outgoing connections
iptables -A OUTPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT

### Allow outbound SSH, Web Traffic
iptables -A OUTPUT -o <output_interface> -p tcp --sport 80 -j ACCEPT
iptables -A OUTPUT -o <output_interface> -p tcp --sport 443 -j ACCEPT
iptables -A OUTPUT -o <output_interface> -p tcp --sport 22 -j ACCEPT

### Allow HTTP/HTTPS traffic to other server (yum install)
iptables -A OUTPUT -o <output_interface> -p tcp --dport 80 -j ACCEPT
iptables -A OUTPUT -o <output_interface> -p tcp --dport 443 -j ACCEPT

### Allow DNS (TCP/UDP port 53), NTP (port 123)
iptables -A OUTPUT -p udp --dport 53 -j ACCEPT
iptables -A OUTPUT -p udp --dport 123 -j ACCEPT

### Block dropped packets
iptables -A OUTPUT -j LOG --log-prefix "IPTables-Dropped: "

### Except the listed above, other connections will be dropped
iptables -t filter -P OUTPUT DROP

#### Block forwarding traffic
## This configuration is for single host set-up, not router
iptables -P FORWARD DROP

#### Save current chain rules (persist after rebooting)
systemctl iptables status
iptables-save > /etc/sysconfig/iptables

#### Reload saved rules (on startup)
iptables-restore < ~/ipt.rules
