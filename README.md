## More About Me – [Take a Look!](http://www.mjakaria.me)

### Welcome to [Prometheus](https://prometheus.io/docs/introduction/overview/)

Prometheus and Grafana are powerful, complementary tools for system monitoring: Prometheus collects and stores real-time metrics, while Grafana visualizes that data in customizable dashboards. Prometheus is the data collector, using its query language (PromQL) to retrieve time-series data, and Grafana connects to Prometheus as a data source to build visual graphs, alerts, and dashboards from that data.

### [Prerequisites](https://training.promlabs.com/training/introduction-to-prometheus/training-overview/prerequisites)

**Prerequisites**

- Ubuntu 24.04 operating system
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
sudo apt upgrade -y
```

- Creating Prometheus System Users and Directory

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
sudo useradd --no-create-home --shell /bin/false node_exporter
```

```bash
sudo mkdir -p /etc/prometheus
sudo mkdir -p /var/lib/prometheus
```

- Give ownership

```bash
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```

- [Download Prometheus Binary File](https://prometheus.io/download/)

```bash
cd /opt/
sudo wget https://github.com/prometheus/prometheus/releases/download/v3.5.0/prometheus-3.5.0.linux-amd64.tar.gz
sudo tar -xvf prometheus-3.5.0.linux-amd64.tar.gz
```

- Copy binaries, consoles and config, and set ownership

```bash
# Adjust folder name if different; tab-complete helps: /opt/prometheus-3.5.0.linux-*
sudo cp /opt/prometheus-3.5.0.linux-amd64/prometheus /usr/local/bin/
sudo cp /opt/prometheus-3.5.0.linux-amd64/promtool /usr/local/bin/

sudo chown prometheus:prometheus /usr/local/bin/prometheus /usr/local/bin/promtool

# copy consoles & config
sudo cp -r /opt/prometheus-3.5.0.linux-amd64/consoles /etc/prometheus
sudo cp -r /opt/prometheus-3.5.0.linux-amd64/console_libraries /etc/prometheus
sudo cp /opt/prometheus-3.5.0.linux-amd64/prometheus.yml /etc/prometheus/

# fix ownership recursively
sudo chown -R prometheus:prometheus /etc/prometheus
ls -la
```

- Edit `prometheus.yml` so it scrapes node_exporter

```bash
sudo tee /etc/prometheus/prometheus.yml > /dev/null <<'EOF'
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
EOF
```

```bash
# ensure ownership
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
```

- Create systemd unit for Prometheus and start it

```bash
sudo tee /etc/systemd/system/prometheus.service > /dev/null <<'EOF'
[Unit]
Description=Prometheus Monitoring
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now prometheus
# verify
sudo systemctl status prometheus --no-pager
# quick check that prometheus is responding
curl -sS http://localhost:9090/-/ready || echo "Prometheus not responding yet"
```

- Install Node Exporter

```bash
cd /opt
sudo wget https://github.com/prometheus/node_exporter/releases/download/v1.10.2/node_exporter-1.10.2.linux-amd64.tar.gz
sudo tar -xvf node_exporter-1.10.2.linux-amd64.tar.gz
sudo cp node_exporter-1.10.2.linux-amd64/node_exporter /usr/local/bin/
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

- Create systemd unit for Node Exporter and start it

```bash
sudo tee /etc/systemd/system/node_exporter.service > /dev/null <<'EOF'
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
sudo systemctl status node_exporter --no-pager
```

```bash
# confirm node_exporter metrics
curl -sS http://localhost:9100/metrics | head -n 20
```

- Tell Prometheus about Node Exporter

```bash
sudo systemctl restart prometheus
# check targets page (on server)
curl -sS http://localhost:9090/api/v1/targets | jq . # if jq installed
# or check in browser: http://SERVER-IP:9090/targets
```

- Open firewall ports

```bash
sudo ufw allow 9090/tcp
sudo ufw allow 3000/tcp
sudo ufw allow 9100/tcp
sudo ufw status
```

- Install Grafana (official repo)

```bash
sudo apt-get install -y apt-transport-https software-properties-common wget gnupg
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt update
sudo apt install grafana -y

sudo systemctl enable --now grafana-server
sudo systemctl status grafana-server --no-pager
```

- Configure Grafana → Add Prometheus as datasource (via web UI)
  - Open browser: <http://SERVER-IP:3000>
  - Login: admin / admin (change password)

- Go to Configuration → Data Sources → Add → choose Prometheus
  - URL: <http://localhost:9090> (or <http://SERVER-IP:9090>)

- Import Grafana dashboard (e.g. ID 14513)
- In Grafana UI → click ‘+’ → Import
- Enter Dashboard ID 14513 → Load → choose your Prometheus data source → Import

- Verify
  - Prometheus UI: <http://SERVER-IP:9090>
  - Check Status → Targets (should show Prometheus & Node Exporter UP)
  - Node Exporter metrics: <http://SERVER-IP:9100/metrics>
  - Grafana Dashboard showing metrics

### [Architecture](https://prometheus.io/docs/introduction/overview/)
![Architecture](/img/architecture.png 'Architecture')

### Terminologies of Prometheus and Grafana

- `Metrics` - Numeric values that represent system performance or resource usage, like `CPU usage`, `memory usage`, `requests per second`.
- `Time Series Data` - Metrics stored with timestamps to show how values change over time, like `CPU usage recorded every second`.
- `PromQL` - Query language used to analyze and retrieve time-series data in Prometheus, like `rate(http_requests_total[5m])`.
- `Exporter` - Component that gathers system metrics and exposes them for Prometheus to scrape, like `Node Exporter`, `MySQL Exporter`.
- `Alerts` - Rules that trigger notifications when specific conditions are met, like `CPU > 90% for 5 minutes`.
- `Target` - Any endpoint from which Prometheus scrapes metrics, like A service exposing `/metrics`.

### PromQL (Prometheus Query Language)

PromQL is the query language used by Prometheus to retrieve, manipulate, and aggregate time series metrics stored in Prometheus.

- It allows you to:
  - Select metrics based on their names and labels.
  - Filter metrics using label matchers.
  - Perform arithmetic and aggregation operations (like sum, average, max, min).
  - Calculate rates, increases, and other functions over time ranges.
  - Create complex queries for monitoring, dashboards, and alerts.

- Key features:
  - Works with time series data (metrics that change over time).
  - Supports instant vectors (current values) and range vectors (values over a time interval).
  - Used in Grafana or Prometheus itself for visualization and alerting.

#### Datatype

1. `Instant Vector:` A set of time series containing a single value for each series at a given time. Used for current metric values.
2. `Range Vector:` A set of time series with values over a range of time. Used for calculations over time, like `rate()` or `increase()`.
3. `Scalar:` A single numeric value, not tied to a specific time series. Often returned by aggregation functions or comparisons.
4. `String:` Rarely used in practice. Represents a string value instead of a number. Mostly used for debugging labels or internal purposes.

#### Summary Table

| Data Type          | Meaning                            | Example Query                         |
| ------------------ | ---------------------------------- | ------------------------------------- |
| **Instant Vector** | Current value for each time series | `node_cpu_seconds_total{mode="idle"}` |
| **Range Vector**   | Values over a time range           | `rate(node_cpu_seconds_total[5m])`    |
| **Scalar**         | Single numeric value               | `count(node_cpu_seconds_total)`       |
| **String**         | Text value                         | Rarely used                           |

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
