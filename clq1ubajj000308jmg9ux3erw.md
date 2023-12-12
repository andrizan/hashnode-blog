---
title: "Store Credential Git"
datePublished: Tue Dec 12 2023 04:26:08 GMT+0000 (Coordinated Universal Time)
cuid: clq1ubajj000308jmg9ux3erw
slug: store-credential-git
canonical: https://stackoverflow.com/a/35942890

---

**Attention**: This method saves the credentials in **plaintext** on your PC's disk. Everyone on your computer can access it, e.g. malicious NPM modules.

Run

```plaintext
git config --global credential.helper store
```

then

```plaintext
git pull
```

provide a username and password and those details will then be remembered later. The credentials are stored in a file on the disk, with the disk permissions of "just user readable/writable" but still in plaintext.

If you want to change the password later

```plaintext
git pull
```

Will fail, because the password is incorrect, git then removes the offending user+password from the `~/.git-credentials` file, so now re-run

```plaintext
git pull
```

to provide a new password so it works as earlier.

Origin = [https://stackoverflow.com/a/35942890](https://stackoverflow.com/a/35942890)