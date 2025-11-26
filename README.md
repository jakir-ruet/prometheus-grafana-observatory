## More About Me – [Take a Look!](http://www.mjakaria.me)

### Welcome to [Prometheus](https://prometheus.io/docs/introduction/overview/)

Prometheus and Grafana are powerful, complementary tools for system monitoring: Prometheus collects and stores real-time metrics, while Grafana visualizes that data in customizable dashboards. Prometheus is the data collector, using its query language (PromQL) to retrieve time-series data, and Grafana connects to Prometheus as a data source to build visual graphs, alerts, and dashboards from that data.

### [Prerequisites](https://training.promlabs.com/training/introduction-to-prometheus/training-overview/prerequisites)

**Prerequisites**

- Ubuntu 20.04 operating system
- Root user account or a user with sudo privileges
- Dedicated Prometheus system user and group
- Adequate storage space and stable internet connectivity
- Required ports:
  - 9090 for Prometheus
  - 3000 for Grafana
  - 9100 for Node Exporter

#### [Prometheus Installation](https://prometheus.io/download/)

- Update the system

```bash
sudo apt update -y
sudo su -
export RELEASE="2.2.1" # Export the release of Prometheus
```

- Creating Prometheus System Users and Directory

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
sudo useradd --no-create-home --shell /bin/false node_exporter
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```

- Give ownership

```bash
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```

- [Download Prometheus Binary File](https://prometheus.io/download/)

```bash
cd /opt/
wget https://github.com/prometheus/prometheus/releases/download/v3.5.0/prometheus-3.5.0.linux-amd64.tar.gz
```

- Install Prometheus on Ubuntu 20.04 LTS

```bash
sha256sum prometheus-2.26.0.linux-amd64.tar.gz # generate a checksum of the Prometheus downloaded file.
tar -xvf prometheus-2.26.0.linux-amd64.tar.gz
cd prometheus-2.26.0.linux-amd64
ls -la
```

- Copy Prometheus Binary files

```bash
sudo cp /opt/prometheus-2.26.0.linux-amd64/prometheus /usr/local/bin/
sudo cp /opt/prometheus-2.26.0.linux-amd64/promtool /usr/local/bin/
```

- Update Prometheus user ownership on Binaries

```bash
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
```

- Copy Prometheus Console Libraries

```bash
sudo cp -r /opt/prometheus-2.26.0.linux-amd64/consoles /etc/prometheus
sudo cp -r /opt/prometheus-2.26.0.linux-amd64/console_libraries /etc/prometheus
sudo cp -r /opt/prometheus-2.26.0.linux-amd64/prometheus.yml /etc/prometheus
```

- Update Prometheus ownership on Directories

```bash
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
sudo chown -R prometheus:prometheus /etc/prometheus/prometheus.yml
```

```bash
prometheus --version
promtool --version
```

- Prometheus configuration file

The `prometheus.yml` file has been copied from `/opt/prometheus-2.26.0.linux-amd64/` to `/etc/prometheus`. Confirm it is present, review its contents, and update it as needed.

```bash
cat /etc/prometheus/prometheus.yml
```

```bash
vi /etc/prometheus/prometheus.yml
- targets: ['localhost:9090', 'localhost:9100'] # update `targets: ['localhost:9090']`
```

- create a system service file in `/etc/systemd/system` location

```bash
sudo vi /etc/systemd/system/prometheus.service
```

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

```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
sudo systemctl status prometheus
```

```bash
sudo ufw allow 9090/tcp # allow firewall
http://server-IP:9090 # login from browser
```

#### [Grafana Installation](https://grafana.com/docs/grafana/latest/setup-grafana/installation/)

```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add –
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
sudo apt-get update
sudo apt-get install grafana
sudo systemctl start grafana-server
sudo systemctl status grafana-server
sudo systemctl enable grafana-server.service
```

```bash
http://server-IP:3000
```

```bash
Username: admin
Password: admin
```

- Configure Prometheus as Grafana DataSource
- Once you logged into Grafana Now first Navigate to `Settings` > `Configuration` > `data sources`
- Add `Data sources` and select `Prometheus`
- Now configure Prometheus data source by providing Prometheus `URL`

#### [Node Exporter Installation](https://prometheus.io/download/) `node_exporter`

- Download and unzip

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.10.2/node_exporter-1.10.2.linux-amd64.tar.gz
sudo tar xvzf node_exporter-1.2.0.linux-amd64.tar.gz
cd node_exporter-1.2.0.linux-amd64
sudo cp node_exporter /usr/local/bin
```

- Creating Node Exporter Systemd service

```bash
cd /lib/systemd/system
sudo vi node_exporter.service
```

```bash
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target
[Service]
Type=simple
User=node_exporter
Group=node_exporter
ExecStart=/usr/local/bin/node_exporter \
— collector.mountstats \
— collector.logind \
— collector.processes \
— collector.ntp \
— collector.systemd \
— collector.tcpstat \
— collector.wifi
Restart=always
RestartSec=10s
[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
sudo systemctl status node_exporter
```

- Configure the Node Exporter as a Prometheus target

```bash
cd /etc/prometheus
sudo vi prometheus.yml
- targets: ['localhost:9090', 'localhost:9100'] # update `targets: ['localhost:9090']`
```

```bash
sudo systemctl restart prometheus
```

```bash
https://localhost:9100/targets
```

- Creating Grafana Dashboard to Monitor Linux Server

We will use dashboard ID `14513` from Grafana.com. Go to the Grafana home page, click the `+` icon, and select `Import`.

[Architecture](https://prometheus.io/docs/introduction/overview/)
![Architecture](/img/architecture.png 'Architecture')

### Terminologies of Prometheus and Grafana

- `Metrics` - Numeric values that represent system performance or resource usage, like `CPU usage`, `memory usage`, `requests per second`.
- `Time Series Data` - Metrics stored with timestamps to show how values change over time, like `CPU usage recorded every second`.
- `PromQL` - Query language used to analyze and retrieve time-series data in Prometheus, like `rate(http_requests_total[5m])`.
- `Exporter` - Component that gathers system metrics and exposes them for Prometheus to scrape, like `Node Exporter`, `MySQL Exporter`.
- `Alerts` - Rules that trigger notifications when specific conditions are met, like `CPU > 90% for 5 minutes`.
- `Target` - Any endpoint from which Prometheus scrapes metrics, like A service exposing `/metrics`.

## With Regards, `Jakir`

[![LinkedIn][linkedin-shield-jakir]][linkedin-url-jakir]
[![Facebook-Page][facebook-shield-jakir]][facebook-url-jakir]
[![Youtube][youtube-shield-jakir]][youtube-url-jakir]

### Wishing you a wonderful day! Keep in touch

<!-- Personal profile -->

[linkedin-shield-jakir]: https://img.shields.io/badge/linkedin-%230077B5.svg?style=for-the-badge&logo=linkedin&logoColor=white
[linkedin-url-jakir]: https://www.linkedin.com/in/jakir-ruet/
[facebook-shield-jakir]: https://img.shields.io/badge/Facebook-%231877F2.svg?style=for-the-badge&logo=Facebook&logoColor=white
[facebook-url-jakir]: https://www.facebook.com/jakir.ruet/
[youtube-shield-jakir]: https://img.shields.io/badge/YouTube-%23FF0000.svg?style=for-the-badge&logo=YouTube&logoColor=white
[youtube-url-jakir]: https://www.youtube.com/@mjakaria-ruet/featured
