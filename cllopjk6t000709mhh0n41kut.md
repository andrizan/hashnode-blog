---
title: "Install Redis 7 Ubuntu"
datePublished: Thu Aug 24 2023 05:12:46 GMT+0000 (Coordinated Universal Time)
cuid: cllopjk6t000709mhh0n41kut
slug: install-redis-7-ubuntu

---

## Install on Ubuntu/Debian

You can install recent stable versions of Redis from the official [`packages.redis.io`](http://packages.redis.io) APT repository.

Prerequisites

If you're running a very minimal distribution (such as a Docker container) you may need to install `lsb-release`, `curl` and `gpg` first:

```bash
sudo apt install lsb-release curl gpg
```

Add the repository to the `apt` index, update it, and then install:

```bash
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

sudo apt-get update
sudo apt-get install redis
```

## Testing Redis

As with any newly-installed software, it’s a good idea to ensure that Redis is functioning as expected before making any further changes to its configuration. We will go over a handful of ways to check that Redis is working correctly in this step.

Start by checking that the Redis service is running:

```bash
sudo systemctl status redis-server
```

## Binding to localhost

By default, Redis is only accessible from [**localhost**](http://localhost). However, if you installed and configured Redis by following a different tutorial than this one, you might have updated the configuration file to allow connections from anywhere. This is not as secure as binding to [**localhost**](http://localhost).

To correct this, open the Redis configuration file for editing:

```bash
sudo nano /etc/redis/redis.conf
```

Locate this line and make sure it is uncommented (remove the `#` if it exists):

```bash
bind 127.0.0.1 ::1
```

Save and close the file when finished (press `CTRL + X`, `Y`, then `ENTER`).

Then, restart the service to ensure that systemd reads your changes:

```bash
sudo systemctl restart redis-server
```

To check that this change has gone into effect, run the following `netstat` command:

```bash
sudo netstat -lnp | grep redis
```

```bash
Output
tcp        0      0 127.0.0.1:6379          0.0.0.0:*               LISTEN      14222/redis-server  
tcp6       0      0 ::1:6379                :::*                    LISTEN      14222/redis-server
```

**Note**: The `netstat` command may not be available on your system by default. If this is the case, you can install it (along with a number of other handy networking tools) with the following command:

```bash
sudo apt install net-tools
```

## Configuring a Redis Password

Configuring a Redis password enables one of its two built-in security features — the `auth` command, which requires clients to authenticate to access the database. The password is configured directly in Redis’s configuration file, `/etc/redis/redis.conf`, so open that file again with your preferred editor:

```bash
sudo nano /etc/redis/redis.conf
```

Scroll to the `SECURITY` section and look for a commented directive that reads:

```bash
. . .
# requirepass foobared
. . .
```

Uncomment it by removing the `#`, and change `foobared` to a secure password.

**Note:** Above the `requirepass` directive in the `redis.conf` file, there is a commented warning:

```bash
. . .
# Warning: since Redis is pretty fast an outside user can try up to
# 150k passwords per second against a good box. This means that you should
# use a very strong password otherwise it will be very easy to break.
#
. . .
```

Thus, it’s important that you specify a very strong and very long value as your password. Rather than make up a password yourself, you can use the `openssl` command to generate a random one, as in the following example. By piping the output of the first command to the second `openssl` command, as shown here, it will remove any line breaks produced by that the first command:

```bash
openssl rand 60 | openssl base64 -A
```

Your output should look something like:

```bash
Output
RBOJ9cCNoGCKhlEBwQLHri1g+atWgn4Xn4HwNUbtzoVxAYxkiYBi7aufl4MILv1nxBqR4L6NNzI0X6cE
```

After copying and pasting the output of that command as the new value for `requirepass`, it should read:

```bash
requirepass RBOJ9cCNoGCKhlEBwQLHri1g+atWgn4Xn4HwNUbtzoVxAYxkiYBi7aufl4MILv1nxBqR4L6NNzI0X6cE
```

After setting the password, save and close the file, then restart Redis:

```bash
sudo systemctl restart redis-server
```

To test that the password works, open up the Redis client:

```bash
redis-cli
```

The following shows a sequence of commands used to test whether the Redis password works. The first command tries to set a key to a value before authentication:

```bash
127.0.0.1:6379> set key1 10
```

That won’t work because you didn’t authenticate, so Redis returns an error:

```bash
Output
127.0.0.1:6379> (error) NOAUTH Authentication required.
```

The next command authenticates with the password specified in the Redis configuration file:

```bash
127.0.0.1:6379> auth your_redis_password
```

Redis acknowledges:

```bash
Output
OK
```

After that, running the previous command again will succeed:

```bash
127.0.0.1:6379> set key1 10
```

```bash
Output
OK
```

`get key1` queries Redis for the value of the new key.

```bash
127.0.0.1:6379> get key1
```

```bash
Output
"10"
```

After confirming that you’re able to run commands in the Redis client after authenticating, you can exit `redis-cli`:

```bash
127.0.0.1:6379> quit
```

src :

* [https://redis.io/docs/getting-started/installation/install-redis-on-linux/](https://redis.io/docs/getting-started/installation/install-redis-on-linux/)
    
* [https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-20-04](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-20-04)