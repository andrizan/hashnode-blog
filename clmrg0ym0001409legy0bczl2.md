---
title: "Setting up Public Key Auth SSH & Allow root"
datePublished: Wed Sep 20 2023 07:49:22 GMT+0000 (Coordinated Universal Time)
cuid: clmrg0ym0001409legy0bczl2
slug: setting-up-public-key-auth-ssh-allow-root

---

configuration file

```bash
nano /etc/ssh/sshd_config
```

change

```bash
#AuthorizedKeysFile      .ssh/authorized_keys .ssh/authorized_keys2

#TO

AuthorizedKeysFile      .ssh/authorized_keys .ssh/authorized_keys2
```

allow root remote

```bash
#PermitRootLogin prohibit-password
# TO
PermitRootLogin yes
```

restart `ssh`

```bash
systemctl restart ssh
```