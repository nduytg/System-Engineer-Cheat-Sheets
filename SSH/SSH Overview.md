# Secure SSH Configuration

- **Author:** nduytg
- **Version:** 1.0
- **Tested on:** CentOS 7

Use strong algorithms, disable password authentication, and enforce key-based
access.

> Audit the configuration before and after hardening with
> <https://github.com/nduytg/ssh-audit/releases/tag/v1.7.0>.

## Generate a key pair

```bash
ssh-keygen -b 4096 -t rsa
ssh-keygen -l -f ~/.ssh/id_rsa.pub
ssh-copy-id user@server
```

## Harden `sshd_config`

```text
Port 44444
Protocol 2
Ciphers aes128-ctr,aes192-ctr,aes256-ctr
HostKeyAlgorithms ecdsa-sha2-nistp256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521,ssh-rsa,ssh-dss
KexAlgorithms ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group14-sha1,diffie-hellman-group-exchange-sha256
MACs hmac-sha2-256,hmac-sha2-512,hmac-sha1
PermitRootLogin no
PasswordAuthentication no
PermitEmptyPasswords no
AllowUsers user1 user2 nduytg
DenyUsers user3 user4
IgnoreRhosts yes
MaxAuthTries 3
MaxSessions 5
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
ClientAliveInterval 300
ClientAliveCountMax 0
AllowTcpForwarding no
AllowStreamLocalForwarding no
GatewayPorts no
PermitTunnel no
```

Use `Match` blocks to override settings for specific users or networks if
required.

## Apply changes

```bash
sudo systemctl restart sshd
```
