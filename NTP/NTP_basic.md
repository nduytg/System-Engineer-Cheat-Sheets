# Configure NTP

- **Author:** nduytg
- **Version:** 1.0
- **Date:** 14/11/17
- **Tested on:** CentOS7

## Reference
https://www.server-world.info/en/note?os=CentOS_7&p=ntp&f=2

### Check Hardware Clock

### Check Software/System/Local Clock
date
hwclock

#### NTP Server
yum –y install ntp
vi /etc/ntp.conf
[…]
restrict 10.0.0.0 mask 255.255.255.0 nomodify notrap
server vn.pool.ntp.org

systemctl start ntpd
systemctl enable ntpd

#### NTP Client

### Option 1: Use NTPd like Server
yum –y install ntp
vi /etc/ntp.conf
[…]
server <server_IP>

systemctl start ntpd
systemctl enable ntpd

### Option 2: Use ntpdate
yum -y install ntpdate

ntpdate -s <domain name/ip>

systemctl enable ntpdate
systemctl start ntpdate
