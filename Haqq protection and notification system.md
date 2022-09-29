# Haqq protection and notification system using Prometheus & Grafana
## Install node_exporter on the 'haqq node' server

```bash
sudo apt-get update && sudo apt-get upgrade -y
```
```bash
cd $HOME
wget https://github.com/prometheus/node_exporter/releases/download/v1.2.0/node_exporter-1.2.0.linux-amd64.tar.gz && \
tar xvf node_exporter-1.2.0.linux-amd64.tar.gz && rm node_exporter-1.2.0.linux-amd64.tar.gz && \
sudo mv node_exporter-1.2.0.linux-amd64 node_exporter && chmod +x $HOME/node_exporter/node_exporter && \
mv $HOME/node_exporter/node_exporter /usr/bin && rm -Rvf $HOME/node_exporter/
```
```bash
sudo tee /etc/systemd/system/exporterd.service > /dev/null <<EOF
[Unit]
Description=node_exporter
After=network-online.target
[Service]
User=$USER
ExecStart=/usr/bin/node_exporter
Restart=always
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
```bash
sudo systemctl daemon-reload && sudo systemctl enable exporterd && sudo systemctl restart exporterd
```
```bash
sudo journalctl -u exporterd -f
``` 
## Install prometheus on new server
```bash
sudo apt-get update && sudo apt-get upgrade -y
```
```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.28.1/prometheus-2.28.1.linux-amd64.tar.gz && \
tar xvf prometheus-2.28.1.linux-amd64.tar.gz && rm prometheus-2.28.1.linux-amd64.tar.gz && mv prometheus-2.28.1.linux-amd64 prometheus
```
```bash
nano $HOME/prometheus/prometheus.yml
#Add node server IP in "targets"
```

```bash
chmod +x $HOME/prometheus/prometheus
```
```bash
sudo tee /etc/systemd/system/prometheusd.service > /dev/null <<EOF
[Unit]
Description=prometheus
After=network-online.target
[Service]
User=$USER
ExecStart=$HOME/prometheus/prometheus --config.file="$HOME/prometheus/prometheus.yml"
Restart=always
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
```bash
sudo systemctl daemon-reload && sudo systemctl enable prometheusd && sudo systemctl restart prometheusd
```
```bash
sudo journalctl -u prometheusd -f
```
## Install grafana on 'prometheus' server
```bash
sudo apt-get install -y adduser libfontconfig1 && wget https://dl.grafana.com/oss/release/grafana_8.0.6_amd64.deb && sudo dpkg -i grafana_8.0.6_amd64.deb
```
```bash
sudo systemctl daemon-reload && sudo systemctl enable grafana-server && sudo systemctl restart grafana-server
```
```bash
sudo journalctl -u grafana-server -f
```
+ Go to http://<grafana_server_IP>:3000/
+ Add Data Source > Prometheus 
+ Specify URL
+ Push Save & test 
+ Import Dashboard
