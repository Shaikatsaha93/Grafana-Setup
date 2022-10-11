# Grafana-Setup step by step

## Install Prometheus Server on Ubuntu 22.04|20.04|18.04

### Step 1: Create Prometheus system group
#### Let’s start by creating the Prometheus system user and group

```
$ sudo groupadd --system prometheus
```
#### The group with ID < 1000 is a system group. Once the system group is added, create Prometheus system user and assign primary group created
```
sudo useradd -s /sbin/nologin --system -g prometheus prometheus
```

### Step 2: Create data & configs directories for Prometheus
#### Prometheus needs a directory to store its data. We will create this under /var/lib/prometheus.
```
sudo mkdir /var/lib/prometheus
```
#### Prometheus primary configuration files directory is /etc/prometheus/. It will have some sub-directories:
```
sudo mkdir -p /etc/prometheus/
```
### Step 3: Download Prometheus on Ubuntu 22.04/20.04/18.04
#### We need to download the latest release of Prometheus archive and extract it to get binary files.
#### Install wget
```
sudo apt update
sudo apt -y install wget curl vim
```
#### Then download latest binary archive for Prometheus
```
mkdir -p /tmp/prometheus && cd /tmp/prometheus
curl -s https://api.github.com/repos/prometheus/prometheus/releases/latest | grep browser_download_url | grep linux-amd64 | cut -d '"' -f 4 | wget -qi -
```
#### Extract the file:
```
tar xvf prometheus*.tar.gz
cd prometheus*/
```
#### Move the binary files to /usr/local/bin/ directory.
```
sudo mv prometheus promtool /usr/local/bin/
```
#### Check installed version:
```
$ prometheus --version
prometheus, version 2.35.0 (branch: HEAD, revision: 6656cd29fe6ac92bab91ecec0fe162ef0f187654)
  build user:       root@cf6852b14d68
  build date:       20220421-09:53:42
  go version:       go1.18.1
  platform:         linux/amd64

$ promtool --version
promtool, version 2.35.0 (branch: HEAD, revision: 6656cd29fe6ac92bab91ecec0fe162ef0f187654)
  build user:       root@cf6852b14d68
  build date:       20220421-09:53:42
  go version:       go1.18.1
  platform:         linux/amd64
  ```
  #### Move Prometheus configuration template to /etc directory.
  ```
  sudo mv prometheus.yml /etc/prometheus/prometheus.yml
  ```
  #### Also move consoles and console_libraries to /etc/prometheus directory:
  ```
  sudo mv consoles/ console_libraries/ /etc/prometheus/
cd $HOME
```

### Step 4: Configure Prometheus on Ubuntu 22.04/20.04/18.04
#### Create or edit a configuration file for Prometheus – /etc/prometheus/prometheus.yml.
```
sudo vim /etc/prometheus/prometheus.yml
```
#### The template configurations should look similar to below:
```
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
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
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:9090']
```
#### You can edit the file to your default liking and save it.

### Create a Prometheus systemd Service unit file
#### To be able to manage Prometheus service with systemd, you need to explicitly define this unit file.
```
sudo tee /etc/systemd/system/prometheus.service<<EOF
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP \$MAINPID
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.external-url=

SyslogIdentifier=prometheus
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```
### Change directory permissions.
#### Change the ownership of these directories to Prometheus user and group.
```
 sudo chown -R prometheus:prometheus /etc/prometheus/
 sudo chmod -R 775 /etc/prometheus/
 sudo chown -R prometheus:prometheus /var/lib/prometheus/
```
#### Reload systemd daemon and start the service:
```
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
```
#### Check status using systemctl status prometheus command:
```
$ systemctl status prometheus
● prometheus.service - Prometheus
   Loaded: loaded (/etc/systemd/system/prometheus.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2020-01-19 14:36:08 UTC; 14s ago
     Docs: https://prometheus.io/docs/introduction/overview/
 Main PID: 1397 (prometheus)
    Tasks: 7 (limit: 2377)
   Memory: 21.7M
   CGroup: /system.slice/prometheus.service
           └─1397 /usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/var/lib/prometheus --web.console.templates

Jan 19 14:36:08 deb10 prometheus[1397]: level=info ts=2020-01-19T14:36:08.959Z caller=main.go:334 vm_limits="(soft=unlimited, hard=unlimited)"
Jan 19 14:36:08 deb10 prometheus[1397]: level=info ts=2020-01-19T14:36:08.960Z caller=main.go:648 msg="Starting TSDB ..."
Jan 19 14:36:08 deb10 prometheus[1397]: level=info ts=2020-01-19T14:36:08.964Z caller=head.go:584 component=tsdb msg="replaying WAL, this may take awhil
Jan 19 14:36:08 deb10 prometheus[1397]: level=info ts=2020-01-19T14:36:08.964Z caller=web.go:506 component=web msg="Start listening for connections" add
Jan 19 14:36:08 deb10 prometheus[1397]: level=info ts=2020-01-19T14:36:08.965Z caller=head.go:632 component=tsdb msg="WAL segment loaded" segment=0 maxS
Jan 19 14:36:08 deb10 prometheus[1397]: level=info ts=2020-01-19T14:36:08.966Z caller=main.go:663 fs_type=EXT4_SUPER_MAGIC
Jan 19 14:36:08 deb10 prometheus[1397]: level=info ts=2020-01-19T14:36:08.966Z caller=main.go:664 msg="TSDB started"
Jan 19 14:36:08 deb10 prometheus[1397]: level=info ts=2020-01-19T14:36:08.966Z caller=main.go:734 msg="Loading configuration file" filename=/etc/prometh
Jan 19 14:36:08 deb10 prometheus[1397]: level=info ts=2020-01-19T14:36:08.967Z caller=main.go:762 msg="Completed loading of configuration file" filename
Jan 19 14:36:08 deb10 prometheus[1397]: level=info ts=2020-01-19T14:36:08.967Z caller=main.go:617 msg="Server is ready to receive web requests."
  ```
  #### If your server has a running firewall service, you’ll need to open port 9090.
  ```
  sudo ufw allow 9090/tcp
  ```
  #### Confirm that you can connect to port 9090 by access the Prometheus server IP address / DNS name in your web browser.
  
  ![Capture](https://user-images.githubusercontent.com/32926005/193406275-286e016c-20c4-47d7-b50e-c55203cb0835.PNG)
  
  
  
  ## How to Install Grafana on Ubuntu 20.04
  ### Log in via SSH and Update your System
  #### First, you will need to log in to your Ubuntu 20.04 VPS via SSH as the root user:
  ```
  ssh root@IP_ADDRESS -p PORT_NUMBER
  ```
  #### Next, run the following commands to upgrade all installed packages on your VPS:
  ```
  apt-get update -y
  ```
  #### Once all the packages are updated, restart your system to apply the changes.
  
  ### Add Grafana Repository
  #### By default, the Grafana package is not included in the Ubuntu 20.04 default repository. So you will need to add the Grafana official repository to your system.
  #### First, install all required dependencies using the following command:
  ```
  apt-get install wget curl gnupg2 apt-transport-https software-properties-common -y
  ```
  #### Next, download and add the Grafana GPG key with the following command:
  ```
  wget -q -O - https://packages.grafana.com/gpg.key | apt-key add -
  ```
  #### Next, add the Grafana repository to APT using the following command:
  ```
  echo "deb https://packages.grafana.com/oss/deb stable main" | tee -a /etc/apt/sources.list.d/grafana.list
  ```
  #### Once the repository is added to your system, you can update it with the following command:
  ```
  apt-get update -y
  ```
  ### Install Grafana
  #### Now, you can install the Grafana by running the following command:
  ```
  apt-get install grafana -y
  ```
  #### Once the Grafana package is installed, verify the Grafana version with the following command:
  ```
  grafana-server -v
  ```
  #### You will get the following output:
  ```
  Version 8.4.5 (commit: 4cafe613e1, branch: HEAD)
  ```
  #### Now, start the Grafana service and enable it to start at system reboot:
  ```
  systemctl start grafana-server
  systemctl enable grafana-server
  ```
  #### You can now check the status of the Grafana with the following command:
  ```
  systemctl status grafana-server
  ```
  #### You will get the following output:
  ```
  ● grafana-server.service - Grafana instance
     Loaded: loaded (/lib/systemd/system/grafana-server.service; disabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-04-06 14:39:27 UTC; 5s ago
       Docs: http://docs.grafana.org
   Main PID: 2761 (grafana-server)
      Tasks: 9 (limit: 2348)
     Memory: 32.2M
     CGroup: /system.slice/grafana-server.service
             └─2761 /usr/sbin/grafana-server --config=/etc/grafana/grafana.ini --pidfile=/run/grafana/grafana-server.pid --packaging=deb cfg:>

Apr 06 14:39:29 ubuntu2004 grafana-server[2761]: logger=sqlstore t=2022-04-06T14:39:29.83+0000 lvl=info msg="Created default admin" user=admin
Apr 06 14:39:29 ubuntu2004 grafana-server[2761]: logger=sqlstore t=2022-04-06T14:39:29.83+0000 lvl=info msg="Created default organization"
Apr 06 14:39:29 ubuntu2004 grafana-server[2761]: logger=plugin.manager t=2022-04-06T14:39:29.89+0000 lvl=info msg="Plugin registered" pluginI>
Apr 06 14:39:29 ubuntu2004 grafana-server[2761]: logger=plugin.finder t=2022-04-06T14:39:29.9+0000 lvl=warn msg="Skipping finding plugins as >
Apr 06 14:39:29 ubuntu2004 grafana-server[2761]: logger=query_data t=2022-04-06T14:39:29.9+0000 lvl=info msg="Query Service initialization"
Apr 06 14:39:29 ubuntu2004 grafana-server[2761]: logger=live.push_http t=2022-04-06T14:39:29.91+0000 lvl=info msg="Live Push Gateway initiali>
Apr 06 14:39:30 ubuntu2004 grafana-server[2761]: logger=server t=2022-04-06T14:39:30.11+0000 lvl=info msg="Writing PID file" path=/run/grafan>
Apr 06 14:39:30 ubuntu2004 grafana-server[2761]: logger=http.server t=2022-04-06T14:39:30.12+0000 lvl=info msg="HTTP Server Listen" address=[>
Apr 06 14:39:30 ubuntu2004 grafana-server[2761]: logger=ngalert t=2022-04-06T14:39:30.13+0000 lvl=info msg="warming cache for startup"
Apr 06 14:39:30 ubuntu2004 grafana-server[2761]: logger=ngalert.multiorg.alertmanager t=2022-04-06T14:39:30.13+0000 lvl=info msg="starting Mu>
```
#### At this point, Grafana is started and listens on port 3000. You can check it with the following command:
```
ss -antpl | grep 3000
```
#### You should see the following output:
```
LISTEN    0         4096                     *:3000                   *:*        users:(("grafana-server",pid=2761,fd=8))  
```

### Configure Nginx as a Reverse Proxy for Grafana
#### Next, you will need to install the Nginx as a reverse proxy for Grafana. First, install the Nginx package using the following command:
```
apt-get install nginx -y
```
#### Once the Nginx is installed, create an Nginx virtual host configuration file:
```
nano /etc/nginx/conf.d/grafana.conf
```
#### Add the following lines:
```
Server {
        server_name grafana.example.com;
        listen 80 ;
        access_log /var/log/nginx/grafana.log;

    location / {
                proxy_pass http://localhost:3000;
        proxy_set_header Host $http_host;
                proxy_set_header X-Forwarded-Host $host:$server_port;
                proxy_set_header X-Forwarded-Server $host;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
}
```
#### Save and close the file then verify the Nginx configuration file using the following command:
```
nginx -t
```
#### You will get the following output:
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
#### Finally, restart the Nginx service to apply the changes:
```
systemctl restart nginx
```

### Access Grafana Dashboard
#### Now, open your web browser and access the Grafana dashboard using the URL ubuntus-erver-IP:3000 You will be redirected to the Grafana login page:

  ![Capture](https://user-images.githubusercontent.com/32926005/195158647-2915e61f-906b-40db-80ec-a926e211bdbf.PNG)

#### Provide default admin username and password as admin/admin and click on the Log in button. You should see the Grafana password change screen:

![installing-grafana-on-ubuntu-20 04](https://user-images.githubusercontent.com/32926005/195160245-2e7c39fd-3b29-4a6c-b501-ebd34f1edbf1.png)

#### Change your default password and click on the Submit button. You should see the Grafana dashboard on the following screen:

![grafana-on-ubuntu-20 04](https://user-images.githubusercontent.com/32926005/195160436-e4b1f5d2-4a23-49ba-87be-7481fc2071cc.png)
