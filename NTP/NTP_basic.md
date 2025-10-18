# Configure NTP on CentOS 7

- **Author:** nduytg
- **Version:** 1.0
- **Date:** 2017-11-14
- **Tested on:** CentOS 7

Synchronize system time using `ntpd` or `ntpdate`.

## Verify clocks

```bash
date
hwclock
```

## NTP server configuration

```bash
sudo yum -y install ntp
sudo sed -i 's/^server .*/server vn.pool.ntp.org/' /etc/ntp.conf
# Allow clients on the local network
printf '\nrestrict 10.0.0.0 mask 255.255.255.0 nomodify notrap\n' | sudo tee -a /etc/ntp.conf
sudo systemctl enable --now ntpd
```

## NTP client options

### Option 1: Run `ntpd`

```bash
sudo yum -y install ntp
sudo bash -c 'cat <<CONF >/etc/ntp.conf
server <server_ip>
CONF'
sudo systemctl enable --now ntpd
```

### Option 2: Periodic sync with `ntpdate`

```bash
sudo yum -y install ntpdate
sudo ntpdate -s <domain-or-ip>
sudo systemctl enable --now ntpdate
```
