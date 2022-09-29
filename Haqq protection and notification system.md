#Install node_exporter on the 'HAQQ node' server

  sudo apt-get update && sudo apt-get upgrade -y
  cd $HOME && wget https://github.com/prometheus/node_exporter/releases/download/v1.2.0/node_exporter-1.2.0.linux-amd64.tar.gz && \
  tar xvf node_exporter-1.2.0.linux-amd64.tar.gz && rm node_exporter-1.2.0.linux-amd64.tar.gz && sudo mv node_exporter-1.2.0.linux-amd64 node_exporter && \
  chmod +x $HOME/node_exporter/node_exporter && mv $HOME/node_exporter/node_exporter /usr/bin && rm -Rvf $HOME/node_exporter/

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

  sudo systemctl daemon-reload && sudo systemctl enable exporterd && sudo systemctl restart exporterd

  sudo journalctl -u exporterd -f
 
#Install prometheus on new server

  sudo apt-get update && sudo apt-get upgrade -y

  wget https://github.com/prometheus/prometheus/releases/download/v2.28.1/prometheus-2.28.1.linux-amd64.tar.gz && \
  tar xvf prometheus-2.28.1.linux-amd64.tar.gz && rm prometheus-2.28.1.linux-amd64.tar.gz && mv prometheus-2.28.1.linux-amd64 prometheus

  nano $HOME/prometheus/prometheus.yml
      #Add node server IP in "targets"

  chmod +x $HOME/prometheus/prometheus

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

  sudo systemctl daemon-reload && \
  sudo systemctl enable prometheusd && \
  sudo systemctl restart prometheusd

  sudo journalctl -u prometheusd -f

#Install grafana on 'prometheus' server

  sudo apt-get install -y adduser libfontconfig1 && wget https://dl.grafana.com/oss/release/grafana_8.0.6_amd64.deb && sudo dpkg -i grafana_8.0.6_amd64.deb

  sudo systemctl daemon-reload && \
  sudo systemctl enable grafana-server && \
  sudo systemctl restart grafana-server

  sudo journalctl -u grafana-server -f

#Go to http://<grafana_server_IP>:3000/ --> add Data Source > Prometheus --> specify URL --> push Save & test --> import Dashboard
