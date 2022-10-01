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
