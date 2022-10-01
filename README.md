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
    
