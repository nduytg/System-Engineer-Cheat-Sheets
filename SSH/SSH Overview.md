# Configure SSH

- **Author:** nduytg

***Note: use this tool to audit SSH server before and after configured
https://github.com/nduytg/ssh-audit/releases/tag/v1.7.0

#### Choosing an algorithm and proper key size
#### Generate key pair
ssh-keygen -b 4096 -t rsa
ssh-keygen -l

### Copy public key to remote server


#### Server-Side Configuration
vi /etc/ssh/sshd_config
[...]

Port 22 (change to the port you want)
Port 44444

## Only use strong cipher algorithm
## Symmetric algorithms
Ciphers aes128-ctr,aes192-ctr,aes256-ctr

## Host Key Algorithms (Key Pair Algorithms)
HostKeyAlgorithms ecdsa-sha2-nistp256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521,ssh-rsa,ssh-dss

## Key Exchange Algorithms
KexAlgorithms ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group14-sha1,diffie-hellman-group-exchange-sha256

## Enable message authentication code algorithms that are used for protecting data
MACs hmac-sha2-256,hmac-sha2-512,hmac-sha1

## Only use SSH Protocol 2
Protocol 2

## Not allow Password + Root login
PermitRootLogin no
PasswordAuthentication no
PermitEmptyPasswords no

## Allow specific users
PermitRootLogin no
AllowUsers user1 user2 nduytg
DenyUsers user3 user4

## Disable rhost file (access to another server base on "trust")
IgnoreRhosts yes

## Limit authentication tries
MaxAuthTries 3
MaxSessions 5


## Allow RSA authentication
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys

## Not allow idle sessions
ClientAliveInterval 300     # 5 minutes
ClientAliveCountMax 0

- Optional: Not allow port forwarding (avoid ssh tunnelling)
AllowTcpForwarding no
AllowStreamLocalForwarding no
GatewayPorts no
PermitTunnel no

## Optional options
## Match Address 192.168.0.1,10.10.123.0/24
then list out your options for these addresses like mentioned above

## Match Users nduytg
then list out your options for these users like mentioned above

---

## Restart sshd service
service sshd restart
