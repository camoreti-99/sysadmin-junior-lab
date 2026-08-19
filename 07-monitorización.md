# 07. Monitorización (Prometheus + Grafana)

---

### 1. **Check VM connectivity (mon01)**

* **How & check:**
  * `"sudo apt update && sudo apt upgrade -y"`
  * `(Screenshot - Monitoring_1)`

---

### 2. **Config node_exporter (mon01 but repeat in fw01, srv01 & web01)**

* **How:**
  * `"cd /tmp" -> "wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz"`
  * `"sudo tar xvfz node_exporter-1.8.2.linux-amd64.tar.gz" -> "sudo cp node_exporter-1.8.2.linux-amd64/node_exporter /usr/local/bin/"`
  * `"sudo useradd --no-create-home --shell /bin/false node_exporter"`
  * `(Screenshot - Node-exporter_1)`
  * `"sudo systemctl daemon-reload"`
  * `"sudo systemctl enable node_exporter"`
  * `"sudo systemctl start node_exporter"`
  * `"sudo systemctl status node_exporter"`

* **Check:**
  * `"curl http://localhost:9100/metrics"`
  * `(Screenshot - Node-exporter_2)`

* **but in fw01:**
  * `sudo apt install wget -y.`
  * `// nft add rule inet filter input iif "ens34" tcp dport 9100 ip saddr 10.10.10.20 accept (Screenshot - Node-exporter_3)`
  * `// previously indicated at nftables.conf (already in), if not, add it`
  * `// nft list ruleset > /etc/nftables.conf`
  * `// nft add rule inet filter forward iif "ens34" oif "ens35" tcp dport 9100 ip saddr 10.10.10.20 accept (Screenshot - Node-exporter_4)`
  * `// previously indicated at nftables.conf (already in), if not, add it`
  * `// nft list ruleset > /etc/nftables.conf`

---

### 3. **Config Prometheus (mon01):**

* **How:**
  * `"cd /tmp" -> "wget https://github.com/prometheus/prometheus/releases/download/v2.54.1/prometheus-2.54.1.linux-amd64.tar.gz"`
  * `"sudo tar xvfz prometheus-2.54.1.linux-amd64.tar.gz"`
  * `"sudo mkdir -p /etc/prometheus"`
  * `"sudo mkdir -p /var/lib/prometheus"`
  * `"sudo cp prometheus-2.54.1.linux-amd64/prometheus /usr/local/bin/"`
  * `"sudo cp prometheus-2.54.1.linux-amd64/promtool /usr/local/bin/"`
  * `"sudo cp -r prometheus-2.54.1.linux-amd64/consoles /etc/prometheus/"`
  * `"sudo cp -r prometheus-2.54.1.linux-amd64/console_libraries /etc/prometheus/"`
  * `"sudo useradd --no-create-home --shell /bin/false prometheus"`
  * `"sudo chown -R prometheus:prometheus /etc/prometheus"`
  * `"sudo chown -R prometheus:prometheus /var/lib/prometheus"`
  * `(Screenshot - Prometheus_1, Prometheus_2, Prometheus_3)`
  * `sudo systemctl daemon-reload`
  * `sudo systemctl enable prometheus`
  * `sudo systemctl start prometheus`
  * `sudo systemctl status prometheus`

* **Check:**
  * `" curl -s 'http://localhost:9090/api/v1/query?query=up' | jq -r '.data.result[] | "\(.metric.job) (\(.metric.instance)): \((.value[1] == "1") | if . then "UP ✅" else "DOWN ❌" end)"' "`
  * `(Screenshot - Prometheus_4, Prometheus_5)`

---

### 4. **Install Grafana (mon01):**

* **How:**
  * `"sudo apt install -y adduser libfontconfig1 musl"`
  * `"wget -q -O - https://packages.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null"`
  * `"echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list"`
  * `"sudo apt update" -> "sudo apt install grafana -y"`
  * `"sudo systemctl enable grafana-server"`
  * `"sudo systemctl start grafana-server"`
  * `"sudo systemctl status grafana-server"`

* **Check:**
  * `(Screenshot - Grafana_1)`

---

### 5. **Config Grafana (cli01):**

* **How:**
  * `admin / admin (change password)`
  * `(Screenshot - Grafana_2, Grafana_3, Grafana_4, Grafana_5, Grafana_6, Grafana_7, Grafana_8)`
  * `Select Prometheus Data Source`
  * `(Screenshot - Grafana_9, Grafana_10)`

* **Check:**
  * `(Screenshot -  Grafana_11)`
