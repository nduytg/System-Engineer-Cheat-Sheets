# SSH Audit Report (Before Hardening)

```text
[0;36m# general[0m
[0;32m(gen) banner: SSH-2.0-OpenSSH_7.4[0m
[0;32m(gen) software: OpenSSH 7.4[0m
[0;32m(gen) compatibility: OpenSSH 7.3+ (some functionality from 6.6), Dropbear SSH 2016.73+ (some functionality from 0.52)[0m
[0;32m(gen) compression: enabled (zlib@openssh.com)[0m

[0;36m# key exchange algorithms[0m
[0;33m(kex) curve25519-sha256                     -- [warn] unknown algorithm[0m
[0;32m(kex) curve25519-sha256@libssh.org          -- [info] available since OpenSSH 6.5, Dropbear SSH 2013.62[0m
[0;31m(kex) ecdh-sha2-nistp256                    -- [fail] using weak elliptic curves[0m
                                            `- [info] available since OpenSSH 5.7, Dropbear SSH 2013.62
[0;31m(kex) ecdh-sha2-nistp384                    -- [fail] using weak elliptic curves[0m
                                            `- [info] available since OpenSSH 5.7, Dropbear SSH 2013.62
[0;31m(kex) ecdh-sha2-nistp521                    -- [fail] using weak elliptic curves[0m
                                            `- [info] available since OpenSSH 5.7, Dropbear SSH 2013.62
[0;33m(kex) diffie-hellman-group-exchange-sha256  -- [warn] using custom size modulus (possibly weak)[0m
                                            `- [info] available since OpenSSH 4.4
[0;32m(kex) diffie-hellman-group16-sha512         -- [info] available since OpenSSH 7.3, Dropbear SSH 2016.73[0m
[0;32m(kex) diffie-hellman-group18-sha512         -- [info] available since OpenSSH 7.3[0m
[0;31m(kex) diffie-hellman-group-exchange-sha1    -- [fail] removed (in server) since OpenSSH 6.7, unsafe algorithm[0m
[0;33m                                            `- [warn] using weak hashing algorithm[0m
                                            `- [info] available since OpenSSH 2.3.0
[0;32m(kex) diffie-hellman-group14-sha256         -- [info] available since OpenSSH 7.3, Dropbear SSH 2016.73[0m
[0;33m(kex) diffie-hellman-group14-sha1           -- [warn] using weak hashing algorithm[0m
                                            `- [info] available since OpenSSH 3.9, Dropbear SSH 0.53
[0;31m(kex) diffie-hellman-group1-sha1            -- [fail] removed (in server) since OpenSSH 6.7, unsafe algorithm[0m
[0;31m                                            `- [fail] disabled (in client) since OpenSSH 7.0, logjam attack[0m
[0;33m                                            `- [warn] using small 1024-bit modulus[0m
[0;33m                                            `- [warn] using weak hashing algorithm[0m
                                            `- [info] available since OpenSSH 2.3.0, Dropbear SSH 0.28

[0;36m# host-key algorithms[0m
[0;32m(key) ssh-rsa                               -- [info] available since OpenSSH 2.5.0, Dropbear SSH 0.28[0m
[0;32m(key) rsa-sha2-512                          -- [info] available since OpenSSH 7.2[0m
[0;32m(key) rsa-sha2-256                          -- [info] available since OpenSSH 7.2[0m
[0;31m(key) ecdsa-sha2-nistp256                   -- [fail] using weak elliptic curves[0m
[0;33m                                            `- [warn] using weak random number generator could reveal the key[0m
                                            `- [info] available since OpenSSH 5.7, Dropbear SSH 2013.62
[0;32m(key) ssh-ed25519                           -- [info] available since OpenSSH 6.5[0m

[0;36m# encryption algorithms (ciphers)[0m
[0;32m(enc) chacha20-poly1305@openssh.com         -- [info] available since OpenSSH 6.5[0m
                                            `- [info] default cipher since OpenSSH 6.9.
[0;32m(enc) aes128-ctr                            -- [info] available since OpenSSH 3.7, Dropbear SSH 0.52[0m
[0;32m(enc) aes192-ctr                            -- [info] available since OpenSSH 3.7[0m
[0;32m(enc) aes256-ctr                            -- [info] available since OpenSSH 3.7, Dropbear SSH 0.52[0m
[0;32m(enc) aes128-gcm@openssh.com                -- [info] available since OpenSSH 6.2[0m
[0;32m(enc) aes256-gcm@openssh.com                -- [info] available since OpenSSH 6.2[0m
[0;31m(enc) aes128-cbc                            -- [fail] removed (in server) since OpenSSH 6.7, unsafe algorithm[0m
[0;33m                                            `- [warn] using weak cipher mode[0m
                                            `- [info] available since OpenSSH 2.3.0, Dropbear SSH 0.28
[0;31m(enc) aes192-cbc                            -- [fail] removed (in server) since OpenSSH 6.7, unsafe algorithm[0m
[0;33m                                            `- [warn] using weak cipher mode[0m
                                            `- [info] available since OpenSSH 2.3.0
[0;31m(enc) aes256-cbc                            -- [fail] removed (in server) since OpenSSH 6.7, unsafe algorithm[0m
[0;33m                                            `- [warn] using weak cipher mode[0m
                                            `- [info] available since OpenSSH 2.3.0, Dropbear SSH 0.47
[0;31m(enc) blowfish-cbc                          -- [fail] removed (in server) since OpenSSH 6.7, unsafe algorithm[0m
[0;31m                                            `- [fail] disabled since Dropbear SSH 0.53[0m
[0;33m                                            `- [warn] disabled (in client) since OpenSSH 7.2, legacy algorithm[0m
[0;33m                                            `- [warn] using weak cipher mode[0m
[0;33m                                            `- [warn] using small 64-bit block size[0m
                                            `- [info] available since OpenSSH 1.2.2, Dropbear SSH 0.28
[0;31m(enc) cast128-cbc                           -- [fail] removed (in server) since OpenSSH 6.7, unsafe algorithm[0m
[0;33m                                            `- [warn] disabled (in client) since OpenSSH 7.2, legacy algorithm[0m
[0;33m                                            `- [warn] using weak cipher mode[0m
[0;33m                                            `- [warn] using small 64-bit block size[0m
                                            `- [info] available since OpenSSH 2.1.0
[0;31m(enc) 3des-cbc                              -- [fail] removed (in server) since OpenSSH 6.7, unsafe algorithm[0m
[0;33m                                            `- [warn] using weak cipher[0m
[0;33m                                            `- [warn] using weak cipher mode[0m
[0;33m                                            `- [warn] using small 64-bit block size[0m
                                            `- [info] available since OpenSSH 1.2.2, Dropbear SSH 0.28

[0;36m# message authentication code algorithms[0m
[0;33m(mac) umac-64-etm@openssh.com               -- [warn] using small 64-bit tag size[0m
                                            `- [info] available since OpenSSH 6.2
[0;32m(mac) umac-128-etm@openssh.com              -- [info] available since OpenSSH 6.2[0m
[0;32m(mac) hmac-sha2-256-etm@openssh.com         -- [info] available since OpenSSH 6.2[0m
[0;32m(mac) hmac-sha2-512-etm@openssh.com         -- [info] available since OpenSSH 6.2[0m
[0;33m(mac) hmac-sha1-etm@openssh.com             -- [warn] using weak hashing algorithm[0m
                                            `- [info] available since OpenSSH 6.2
[0;33m(mac) umac-64@openssh.com                   -- [warn] using encrypt-and-MAC mode[0m
[0;33m                                            `- [warn] using small 64-bit tag size[0m
                                            `- [info] available since OpenSSH 4.7
[0;33m(mac) umac-128@openssh.com                  -- [warn] using encrypt-and-MAC mode[0m
                                            `- [info] available since OpenSSH 6.2
[0;33m(mac) hmac-sha2-256                         -- [warn] using encrypt-and-MAC mode[0m
                                            `- [info] available since OpenSSH 5.9, Dropbear SSH 2013.56
[0;33m(mac) hmac-sha2-512                         -- [warn] using encrypt-and-MAC mode[0m
                                            `- [info] available since OpenSSH 5.9, Dropbear SSH 2013.56
[0;33m(mac) hmac-sha1                             -- [warn] using encrypt-and-MAC mode[0m
[0;33m                                            `- [warn] using weak hashing algorithm[0m
                                            `- [info] available since OpenSSH 2.1.0, Dropbear SSH 0.28

[0;36m# algorithm recommendations (for OpenSSH 7.4)[0m
[0;33m(rec) -diffie-hellman-group14-sha1          -- kex algorithm to remove [0m
[0;31m(rec) -ecdh-sha2-nistp256                   -- kex algorithm to remove [0m
[0;33m(rec) -diffie-hellman-group-exchange-sha256 -- kex algorithm to remove [0m
[0;31m(rec) -diffie-hellman-group1-sha1           -- kex algorithm to remove [0m
[0;31m(rec) -diffie-hellman-group-exchange-sha1   -- kex algorithm to remove [0m
[0;31m(rec) -ecdh-sha2-nistp521                   -- kex algorithm to remove [0m
[0;31m(rec) -ecdh-sha2-nistp384                   -- kex algorithm to remove [0m
[0;31m(rec) -ecdsa-sha2-nistp256                  -- key algorithm to remove [0m
[0;31m(rec) -blowfish-cbc                         -- enc algorithm to remove [0m
[0;31m(rec) -3des-cbc                             -- enc algorithm to remove [0m
[0;31m(rec) -aes256-cbc                           -- enc algorithm to remove [0m
[0;31m(rec) -cast128-cbc                          -- enc algorithm to remove [0m
[0;31m(rec) -aes192-cbc                           -- enc algorithm to remove [0m
[0;31m(rec) -aes128-cbc                           -- enc algorithm to remove [0m
[0;33m(rec) -hmac-sha2-512                        -- mac algorithm to remove [0m
[0;33m(rec) -umac-128@openssh.com                 -- mac algorithm to remove [0m
[0;33m(rec) -hmac-sha2-256                        -- mac algorithm to remove [0m
[0;33m(rec) -umac-64@openssh.com                  -- mac algorithm to remove [0m
[0;33m(rec) -hmac-sha1                            -- mac algorithm to remove [0m
[0;33m(rec) -hmac-sha1-etm@openssh.com            -- mac algorithm to remove [0m
[0;33m(rec) -umac-64-etm@openssh.com              -- mac algorithm to remove [0m


```
