# prometheus-monitoring

## Install Grafana
```
wget https://dl.grafana.com/oss/release/grafana-7.3.1-1.x86_64.rpm
yum install grafana-7.3.1-1.x86_64.rpm
```

## Install Prometheus
```
# refer: https://www.howtoforge.com/tutorial/how-to-install-prometheus-and-node-exporter-on-centos-7/
wget https://github.com/prometheus/prometheus/releases/download/v2.22.1/prometheus-2.22.1.linux-amd64.tar.gz
useradd --no-create-home --shell /bin/false prometheus
mkdir /etc/prometheus
mkdir /var/lib/prometheus
chown prometheus:prometheus /etc/prometheus
chown prometheus:prometheus /var/lib/prometheus
tar -zxvf prometheus-2.22.1.linux-amd64.tar.gz 
mv prometheus-2.22.1.linux-amd64 prometheuspackage
cp prometheuspackage/prometheus /usr/local/bin/
cp prometheuspackage/promtool /usr/local/bin/
chown prometheus:prometheus /usr/local/bin/prometheus
chown prometheus:prometheus /usr/local/bin/promtool 
cp -r prometheuspackage/consoles /etc/prometheus
cp -r prometheuspackage/console_libraries /etc/prometheus
chown -R prometheus:prometheus /etc/prometheus/consoles
chown -R prometheus:prometheus /etc/prometheus/console_libraries
chown prometheus:prometheus /etc/prometheus/prometheus.yml
chown prometheus:prometheus /etc/prometheus/prometheus.yml
vim /etc/systemd/system/prometheus.service

#### cat /etc/systemd/system/prometheus.service 
#----------------------EOF
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
--web.console.libraries=/etc/prometheus/console_libraries \
--web.enable-admin-api

[Install]
WantedBy=multi-user.target
#----------------------EOF
```

## delete old prometheus data
```
curl -s -X POST -g 'http://localhost:9090/api/v1/admin/tsdb/clean_tombstones'
```

## restart prometheus
```
vim /etc/prometheus/prometheus.yml
sudo service prometheus restart
```

## node exporter
```
# refer: https://www.howtoforge.com/tutorial/how-to-install-prometheus-and-node-exporter-on-centos-7/
wget https://github.com/prometheus/node_exporter/releases/download/v1.0.1/node_exporter-1.0.1.linux-amd64.tar.gz
tar xvfz node_exporter-1.0.1.linux-amd64.tar.gz 
cd node_exporter-1.0.1.linux-amd64

cp -r node_exporter /usr/local/bin/
#### vim /etc/systemd/system/node_exporter.service
#----------------------EOF
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target
[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter
[Install]
WantedBy=multi-user.target
#----------------------EOF
 
ls /usr/local/bin/node_exporter
useradd --no-create-home --shell /bin/false node_exporter
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter 
cd /etc/systemd/system/
vim node_exporter.service
systemctl daemon-reload
systemctl start node_exporter
systemctl status node_exporter
history
```

## install blackbox 
```
# refer: https://devconnected.com/how-to-install-and-configure-blackbox-exporter-for-prometheus/
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.18.0/blackbox_exporter-0.18.0.linux-amd64.tar.gz
tar -zxvf blackbox_exporter-0.18.0.linux-amd64.tar.gz 
cd blackbox_exporter-0.18.0.linux-amd64
sudo mv blackbox_exporter /usr/local/bin
sudo mkdir -p /etc/blackbox
sudo mv blackbox.yml /etc/blackbox
sudo useradd --no-create-home -rs /bin/false blackbox
sudo chown blackbox:blackbox /usr/local/bin/blackbox_exporter
sudo chown -R blackbox:blackbox /etc/blackbox/*

# vim /lib/systemd/system/blackbox.service
#----------------------EOF
[Unit]
Description=Blackbox Exporter Service
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=blackbox
Group=blackbox
ExecStart=/usr/local/bin/blackbox_exporter \
  --config.file=/etc/blackbox/blackbox.yml \
  --web.listen-address=":9115"

Restart=always

[Install]
WantedBy=multi-user.target
#----------------------EOF

systemctl daemon-reload
sudo systemctl enable blackbox.service
sudo systemctl start blackbox.service
```
