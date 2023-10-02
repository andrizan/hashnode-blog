---
title: "SCP Linux"
datePublished: Mon Oct 02 2023 08:05:25 GMT+0000 (Coordinated Universal Time)
cuid: cln8lvtnq000109l67phvhj97
slug: scp-linux

---

Syntax:

```bash
scp <source> <destination>
```

To copy a file from `B` to `A` while logged into `B`:

```bash
scp /path/to/file username@a:/path/to/destination
```

To copy a file from `B` to `A` while logged into `A`:

```bash
scp username@b:/path/to/file /path/to/destination
```

To copy a file from `B` to `A` while logged into `B`. NON STANDARD PORT:

```bash
scp -P 2200 /path/to/file username@a:/path/to/destination
```

[https://unix.stackexchange.com/questions/106480/how-to-copy-files-from-one-machine-to-another-using-ssh](https://unix.stackexchange.com/questions/106480/how-to-copy-files-from-one-machine-to-another-using-ssh)