# Prometheus Installation Guide (Production-Ready)

This guide provides step-by-step instructions to install Prometheus on a Linux server for a real company environment. You can use this file in your GitHub repository.

---

# Grafana Installation Guide (Production-Ready)

## 1. Install Grafana (Ubuntu/Debian Example)

```sh
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt-get update
sudo apt-get install grafana
```

## 2. Start and Enable Grafana

```sh
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

## 3. Access Grafana

- Open your browser and go to: `http://<your-server-ip>:3000`
- Default login: `admin` / `admin` (you’ll be prompted to change the password)

## 4. Add Prometheus as a Data Source

1. Go to **Configuration → Data Sources** in Grafana UI.
2. Click **Add data source** and select **Prometheus**.
3. Set the URL to `http://localhost:9090` (or your Prometheus server address).
4. Click **Save & Test**.

## 5. Provision Prometheus Data Source Automatically (Optional)

Create a file named `grafana-prometheus-datasource.yaml` in `/etc/grafana/provisioning/datasources/` with:

```yaml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://localhost:9090
    isDefault: true
```

## 6. Import Dashboards

- Go to **Create → Import** in Grafana UI.
- Use dashboard IDs from [Grafana.com Dashboards](https://grafana.com/grafana/dashboards/).

---

## 1. Prepare the Server
- Use a dedicated Linux VM or server (Ubuntu/CentOS recommended).
- Ensure you have root or sudo access.

## 2. Create a Prometheus User
```sh
sudo useradd --no-create-home --shell /bin/false prometheus
```

## 3. Create Directories
```sh
sudo mkdir /etc/prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /etc/prometheus /var/lib/prometheus
```

## 4. Download Prometheus
- Go to the [Prometheus releases page](https://prometheus.io/download/).
- Download the latest stable version (replace version if newer is available):
```sh
wget https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-amd64.tar.gz
tar xvf prometheus-2.52.0.linux-amd64.tar.gz
cd prometheus-2.52.0.linux-amd64
```

## 5. Move Binaries
```sh
sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus /usr/local/bin/promtool
```

## 6. Move Configuration Files
```sh
sudo cp -r consoles /etc/prometheus
sudo cp -r console_libraries /etc/prometheus
sudo cp prometheus.yml /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus
```

## 7. Create a Systemd Service
Create `/etc/systemd/system/prometheus.service` with the following content:
```ini
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
  --storage.tsdb.path=/var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

## 8. Start and Enable Prometheus
```sh
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
```

## 9. Verify Installation
- Check status: `sudo systemctl status prometheus`
- Open browser: `http://<server-ip>:9090`

## 10. Secure Prometheus (Production)
- Restrict firewall to trusted IPs.
- Use a reverse proxy (Nginx/Apache) for authentication and TLS.
- Regularly backup configuration and data.

---

**For automation or advanced configuration (alerts, exporters, dashboards), see the official [Prometheus documentation](https://prometheus.io/docs/introduction/overview/).**
