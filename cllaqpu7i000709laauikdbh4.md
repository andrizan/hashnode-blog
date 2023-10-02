---
title: "Deploy MinIO Ubuntu Server: Single-Node Single-Drive"
datePublished: Mon Aug 14 2023 10:36:52 GMT+0000 (Coordinated Universal Time)
cuid: cllaqpu7i000709laauikdbh4
slug: deploy-minio-ubuntu-server-single-node-single-drive
tags: ubuntu, minio

---

# Download the MinIO Server

The following tabs provide examples of installing MinIO onto 64-bit Linux operating systems using RPM, DEB, or binary. The RPM and DEB packages automatically install MinIO to the necessary system paths and create a `minio` service for `systemctl`. MinIO strongly recommends using the RPM or DEB installation routes. To update deployments managed using `systemctl`, see [Update systemctl-Managed MinIO Deployments](https://min.io/docs/minio/linux/operations/install-deploy-manage/upgrade-minio-deployment.html#minio-upgrade-systemctl).

## AMD64

Use the following commands to download the latest stable MinIO binary and install it to the system `$PATH`:

```bash
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
sudo mv minio /usr/local/bin/
```

# Create the `systemd` Service File

The `.deb` or `.rpm` packages install the following [systemd](https://www.freedesktop.org/wiki/Software/systemd/) service file to `/etc/systemd/system/minio.service`. For binary installations, create this file manually on all MinIO hosts:

```bash
[Unit]
Description=MinIO
Documentation=https://min.io/docs/minio/linux/index.html
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/usr/local/bin/minio

[Service]
WorkingDirectory=/usr/local

User=minio-user
Group=minio-user
ProtectProc=invisible

EnvironmentFile=-/etc/default/minio
ExecStartPre=/bin/bash -c "if [ -z \"${MINIO_VOLUMES}\" ]; then echo \"Variable MINIO_VOLUMES not set in /etc/default/minio\"; exit 1; fi"
ExecStart=/usr/local/bin/minio server $MINIO_OPTS $MINIO_VOLUMES

# MinIO RELEASE.2023-05-04T21-44-30Z adds support for Type=notify (https://www.freedesktop.org/software/systemd/man/systemd.service.html#Type=)
# This may improve systemctl setups where other services use `After=minio.server`
# Uncomment the line to enable the functionality
# Type=notify

# Let systemd restart this service always
Restart=always

# Specifies the maximum file descriptor number that can be opened by this process
LimitNOFILE=65536

# Specifies the maximum number of threads this process can create
TasksMax=infinity

# Disable timeout logic and wait until process is stopped
TimeoutStopSec=infinity
SendSIGKILL=no

[Install]
WantedBy=multi-user.target

# Built for ${project.name}-${project.version} (${project.name})
```

The `minio.service` file runs as the `minio-user` User and Group by default. You can create the user and group using the `groupadd` and `useradd` commands. The following example creates the user, group, and sets permissions to access the folder paths intended for use by MinIO. These commands typically require root (`sudo`) permissions.

```bash
groupadd -r minio-user
useradd -M -r -g minio-user minio-user
chown minio-user:minio-user /mnt/disk1 /mnt/disk2 /mnt/disk3 /mnt/disk4
```

The specified drive paths are provided as an example. Change them to match the path to those drives intended for use by MinIO.

Alternatively, change the `User` and `Group` values to another user and group on the system host with the necessary access and permissions.

MinIO publishes additional startup script examples on [github.com/minio/minio-service](http://github.com/minio/minio-service).

To update deployments managed using `systemctl`, see [Update systemctl-Managed MinIO Deployments](https://min.io/docs/minio/linux/operations/install-deploy-manage/upgrade-minio-deployment.html#minio-upgrade-systemctl).

# Create the Environment Variable File

Create an environment variable file at `/etc/default/minio`. For Windows hosts, specify a Windows-style path similar to `C:\minio\config`. The MinIO Server container can use this file as the source of all [environment variables](https://min.io/docs/minio/linux/reference/minio-server/minio-server.html#minio-server-environment-variables).

The following example provides a starting environment file:

```bash
# MINIO_ROOT_USER and MINIO_ROOT_PASSWORD sets the root account for the MinIO server.
# This user has unrestricted permissions to perform S3 and administrative API operations on any resource in the deployment.
# Omit to use the default values 'minioadmin:minioadmin'.
# MinIO recommends setting non-default values as a best practice, regardless of environment

MINIO_ROOT_USER=myminioadmin
MINIO_ROOT_PASSWORD=minio-secret-key-change-me

# MINIO_VOLUMES sets the storage volume or path to use for the MinIO server.

MINIO_VOLUMES="/mnt/data"

# MINIO_SERVER_URL sets the hostname of the local machine for use with the MinIO Server
# MinIO assumes your network control plane can correctly resolve this hostname to the local machine

# Uncomment the following line and replace the value with the correct hostname for the local machine and port for the MinIO server (9000 by default).

#MINIO_SERVER_URL="http://ip:9060"
#MINIO_CONSOLE_ADDRESS=":9000"

# Uncomment the if use domain
#MINIO_BROWSER_REDIRECT_URL="http://s3.domain.com/minio/ui"

#MINIO_PROMETHEUS_URL=http://192.168.100.16:9090
#MINIO_PROMETHEUS_JOB_ID="minio-job"
```

Include any other environment variables as required for your local deployment.

## Nginx (reverse proxy)

install nginx

```bash
server {
   server_name s3.domain.com;
   listen 80;

   # Allow special characters in headers
   ignore_invalid_headers off;
   # Allow any size file to be uploaded.
   # Set to a value such as 1000m; to restrict file size to a specific value
   client_max_body_size 0;
   # Disable buffering
   proxy_buffering off;
   proxy_request_buffering off;

   location / {
      proxy_set_header Host $http_host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;

      proxy_connect_timeout 300;
      # Default is HTTP/1, keepalive is only enabled in HTTP/1.1
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      chunked_transfer_encoding off;

      proxy_pass http://127.0.0.1:9000; # This uses the upstream directive definition to load balance
   }

   location /minio/ui/ {
      rewrite ^/minio/ui/(.*) /$1 break;
      proxy_set_header Host $http_host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
      proxy_set_header X-NginX-Proxy true;

      # This is necessary to pass the correct IP to be hashed
      real_ip_header X-Real-IP;

      proxy_connect_timeout 300;

      # To support websockets in MinIO versions released after January 2023
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      # Some environments may encounter CORS errors (Kubernetes + Nginx Ingress)
      # Uncomment the following line to set the Origin request to an empty string
      # proxy_set_header Origin '';

      chunked_transfer_encoding off;

      proxy_pass http://127.0.0.1:9001/;
   }

}
```

# Start the MinIO Service

Issue the following command on the local host to start the MinIO <abbr>SNSD</abbr> deployment as a service:

```bash
sudo systemctl start minio.service
```

Use the following commands to confirm the service is online and functional:

```bash
sudo systemctl status minio.service
journalctl -f -u minio.service
```

MinIO may log an increased number of non-critical warnings while the server processes connect and synchronize. These warnings are typically transient and should resolve as the deployment comes online.

Changed in version RELEASE.2023-02-09T05-16-53Z: MinIO starts if it detects enough drives to meet the [write quorum](https://min.io/docs/minio/linux/operations/concepts/erasure-coding.html#minio-ec-parity) for the deployment.

If any drives remain offline after starting MinIO, check and cure any issues blocking their functionality before starting production workloads.

The `journalctl` output should resemble the following:

```bash
Status:         1 Online, 0 Offline.
API: http://192.168.2.100:9000  http://127.0.0.1:9000
RootUser: myminioadmin
RootPass: minio-secret-key-change-me
Console: http://192.168.2.100:9090 http://127.0.0.1:9090
RootUser: myminioadmin
RootPass: minio-secret-key-change-me

Command-line: https://min.io/docs/minio/linux/reference/minio-mc.html
   $ mc alias set myminio http://10.0.2.100:9000 myminioadmin minio-secret-key-change-me

Documentation: https://min.io/docs/minio/linux/index.html
```

The `API` block lists the network interfaces and port on which clients can access the MinIO S3 API. The `Console` block lists the network interfaces and port on which clients can access the MinIO Web Console.

# Connect to the MinIO Service

You can access the MinIO Console by entering any of the hostnames or IP addresses from the MinIO server `Console` block in your preferred browser, such as [http://localhost:9090](http://localhost:9090).

Log in with the [`MINIO_ROOT_USER`](https://min.io/docs/minio/linux/reference/minio-server/minio-server.html#envvar.MINIO_ROOT_USER) and [`MINIO_ROOT_PASSWORD`](https://min.io/docs/minio/linux/reference/minio-server/minio-server.html#envvar.MINIO_ROOT_PASSWORD) configured in the environment file specified to the container.

![MinIO Console displaying Buckets view in a fresh installation](https://min.io/docs/minio/linux/_images/console-bucket-none.png align="left")

You can use the MinIO Console for general administration tasks like Identity and Access Management, Metrics and Log Monitoring, or Server Configuration. Each MinIO server includes its own embedded MinIO Console.

If your local host firewall permits external access to the Console port, other hosts on the same network can access the Console using the IP or hostname for your local host.

## Cloudflared Tunnel

Disable `cache` to resolve error `403`

`dash.cloudflare.com > caching > cache-rules`

rule :

* hostname = DOMAIN/sub Domain
    
* Cache Status = Bypass Cache
    

# Install Prometheus on Ubuntu

## Update System Packages

You should first update your system's package list to ensure that you are using the most recent packages. To accomplish this, issue the following command:

```bash
sudo apt update
```

## Create a System User for Prometheus

Now create a group and a system user for Prometheus. To create a group and then add a user to the group, run the following command:

```bash
sudo groupadd --system prometheus
sudo useradd -s /sbin/nologin --system -g prometheus prometheus
```

![add group and user](https://www.cherryservers.com/v3/assets/blog/2023-05-16/01.png align="left")

This will create a system user and group named "prometheus" for Prometheus with limited privileges, reducing the risk of unauthorized access.

## Create Directories for Prometheus

To store configuration files and libraries for Prometheus, you need to create a few directories. The directories will be located in the `/etc` and the `/var/lib` directory respectively. Use the commands below to create the directories:

```bash
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```

![create directories for Prometheus](https://www.cherryservers.com/v3/assets/blog/2023-05-16/03.png align="left")

## Download Prometheus and Extract Files

To download the latest update, go to the [Prometheus official downloads site](https://prometheus.io/download/#prometheus) and copy the download link for Linux Operating System. Download using wget and the link you copied like so:

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.43.0/prometheus-2.43.0.linux-amd64.tar.gz
```

You should see it being downloaded.

![download Prometheus](https://www.cherryservers.com/v3/assets/blog/2023-05-16/04.png align="left")

After the download has been completed, run the following command to extract the contents of the downloaded file:

```bash
tar vxf prometheus*.tar.gz
```

![extract Prometheus file](https://www.cherryservers.com/v3/assets/blog/2023-05-16/05.png align="left")

## Navigate to the Prometheus Directory

After extracting the files, navigate to the newly extracted Prometheus directory using the following command:

```bash
cd prometheus*/
```

![change directory](https://www.cherryservers.com/v3/assets/blog/2023-05-16/06.png align="left")

Changing to the Prometheus directory allows for easier management and configuration of the installation. Subsequent steps will be performed within the context of the Prometheus directory.

# Configuring Prometheus on Ubuntu 22.04

## Move the Binary Files & Set Owner

First, you need to move some binary files (**prometheus** and **promtool**) and change the ownership of the files to the "**prometheus"** user and group. You can do this with the following commands:

```bash
sudo mv prometheus /usr/local/bin
sudo mv promtool /usr/local/bin
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
```

![move binary files and set owner](https://www.cherryservers.com/v3/assets/blog/2023-05-16/07.png align="left")

## Move the Configuration Files & Set Owner

Create `minio-alerting.yml`

```yaml
# minio-alerting.yml 
groups:
- name: minio-alerts
  rules:
  - alert: NodesOffline
    expr: avg_over_time(minio_cluster_nodes_offline_total{job="minio-job"}[5m]) > 0
    for: 10m
    labels:
      severity: warn
    annotations:
      summary: "Node down in MinIO deployment"
      description: "Node(s) in cluster {{ $labels.instance }} offline for more than 5 minutes"

  - alert: DisksOffline
    expr: avg_over_time(minio_cluster_disk_offline_total{job="minio-job"}[5m]) > 0
    for: 10m
    labels:
      severity: warn
    annotations:
      summary: "Disks down in MinIO deployment"
      description: "Disks(s) in cluster {{ $labels.instance }} offline for more than 5 minutes"
```

change `prometheus.yml`

```yaml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - minio-alerting.yml
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  - job_name: minio-job
    bearer_token: <TOKEN> # https://min.io/docs/minio/linux/reference/minio-mc-admin/mc-admin-prometheus.html | https://min.io/docs/minio/linux/reference/minio-mc.html
    metrics_path: /minio/v2/metrics/cluster
    scheme: http
    static_configs:
      - targets: ['192.168.100.14:9000']

  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
```

how to get tokens can be read here [https://min.io/docs/minio/linux/reference/minio-mc-admin/mc-admin-prometheus.html](https://min.io/docs/minio/linux/reference/minio-mc-admin/mc-admin-prometheus.html)

install `mc-cli` binary [https://min.io/docs/minio/linux/reference/minio-mc.html](https://min.io/docs/minio/linux/reference/minio-mc.html)

Next, move the configuration files and set their ownership so that Prometheus can access them. To do this, run the following commands:

```bash
sudo mv consoles /etc/prometheus
sudo mv console_libraries /etc/prometheus
sudo mv prometheus.yml /etc/prometheus
sudo mv minio-alerting.yml /etc/prometheus
```

```bash
sudo chown prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
sudo chown -R prometheus:prometheus /var/lib/prometheus
```

![move configuration files and set owner](https://www.cherryservers.com/v3/assets/blog/2023-05-16/08.png align="left")

The `prometheus.yml` file is the main Prometheus configuration file. It includes settings for targets to be monitored, data scraping frequency, data processing, and storage. You can set alerting rules and notification conditions in the file. You don't need to modify this file for this demonstration but feel free to open it in an editor to take a closer look at its contents.

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Here's the default content of the Prometheus file:

![Prometheus file content](https://www.cherryservers.com/v3/assets/blog/2023-05-16/09.png align="left")

![Prometheus file content](https://www.cherryservers.com/v3/assets/blog/2023-05-16/10.png align="left")

![Prometheus file content](https://www.cherryservers.com/v3/assets/blog/2023-05-16/11.png align="left")

## Create Prometheus Systemd Service

Now, you need to create a system service file for Prometheus. Create and open a `prometheus.service` file with the Nano text editor using:

```bash
sudo nano /etc/systemd/system/prometheus.service
```

Include these settings to the file, save, and exit:

```bash
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

**Alternative Config**

```bash
[Unit]
Description=Prometheus #Description
Documentation=https://prometheus.io/docs/introduction/overview/ #reference to documentation
Wants=network-online.target
After=network-online.target
[Service]
Type=simple
User=prometheus #user
Group=prometheus #group
ExecReload=/bin/kill -HUP \$MAINPID
ExecStart=/usr/local/bin/prometheus \
--config.file=/etc/prometheus/prometheus.yml \ #main config
--storage.tsdb.path=/var/lib/prometheus \ #database
--web.console.templates=/etc/prometheus/consoles \
--web.console.libraries=/etc/prometheus/console_libraries \
--web.listen-address=0.0.0.0:9090 \
--web.external-url=
SyslogIdentifier=prometheus #name of log file
Restart=always #enable restart
[Install]
WantedBy=multi-user.target
```

![Prometheus systemd service file](https://www.cherryservers.com/v3/assets/blog/2023-05-16/12.png align="left")

![Prometheus systemd service file](https://www.cherryservers.com/v3/assets/blog/2023-05-16/13.png align="left")

The "systems" service file for Prometheus defines how Prometheus should be managed as a system service on Ubuntu. It includes the service configuration, such as the user and group it should run as. It also includes the path to the Prometheus binary and the Prometheus configuration file location. Additionally, the file can be used to set storage locations for metrics data and pass additional command-line options to the Prometheus binary when it starts.

## Reload Systemd

You need to reload the system configuration files after saving the `prometheus.service` file so that changes made are recognized by the system. Reload the system configuration files using the following:

```bash
sudo systemctl daemon-reload
```

![Reload systemd](https://www.cherryservers.com/v3/assets/blog/2023-05-16/14.png align="left")

## Start Prometheus Service

Next, you want to enable and start your Prometheus service. Do this using the following commands:

```bash
sudo systemctl enable prometheus
sudo systemctl start prometheus
```

![Start Prometheus service](https://www.cherryservers.com/v3/assets/blog/2023-05-16/15.png align="left")

## Check Prometheus Status

After starting the Prometheus service, you may confirm that it is running or if you have encountered errors using:

```bash
sudo systemctl status prometheus
```

Sample output:

![Check Prometheus status](https://www.cherryservers.com/v3/assets/blog/2023-05-16/16.png align="left")

# Access Prometheus Web Interface

Prometheus runs on port 9090 by default so you need to allow port 9090 on your firewall, Do that using the command:

```bash
sudo ufw allow 9090/tcp
```

![Allow port 9090](https://www.cherryservers.com/v3/assets/blog/2023-05-16/17.png align="left")

With Prometheus running successfully, you can access it via your web browser using [localhost:9090](http://localhost:9090) or &lt;ip\_address&gt;:9090

src

* [https://min.io/docs/minio/linux/operations/install-deploy-manage/deploy-minio-single-node-single-drive.html#minio-snsd](https://min.io/docs/minio/linux/operations/install-deploy-manage/deploy-minio-single-node-single-drive.html#minio-snsd)
    
* [https://www.cherryservers.com/blog/install-prometheus-ubuntu](https://www.cherryservers.com/blog/install-prometheus-ubuntu)