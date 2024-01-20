Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-22.04"

  # MySQL
  config.vm.define "mysql" do |mysql_config|
    mysql_config.vm.network "public_network", ip: "192.168.1.2", netmask: "255.255.255.0", bridge: "eth1"
    mysql_config.vm.hostname = "mysql"

    mysql_config.vm.provider "virtualbox" do |v|
      v.memory = 4096
      v.cpus = 2
    end

    mysql_config.vm.provision "shell", inline: <<-SHELL
      sudo apt update && sudo apt upgrade -y
      
      sudo apt install mysql-server -y

      echo "CREATE DATABASE Users;" | mysql -uroot
      echo "GRANT ALL PRIVILEGES ON Users.* TO 'user'@'localhost' IDENTIFIED BY 'password';" | mysql -uroot
      echo "FLUSH PRIVILEGES;" | mysql -uroot
  
      wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
      tar xvfz node_exporter-*.tar.gz
      sudo mv node_exporter-1.5.0.linux-amd64/node_exporter /usr/local/bin
      rm -r node_exporter-1.5.0.linux-amd64*

      sudo useradd -rs /bin/false node_exporter
      sudo chown -R node_exporter: /usr/local/bin/node_exporter

      sudo tee /etc/systemd/system/node_exporter.service > /dev/null <<EOF
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF

      sudo systemctl daemon-reload
      sudo systemctl enable node_exporter
      sudo systemctl start node_exporter
    SHELL
  end

  # Grafana
  config.vm.define "grafana" do |grafana_config|
    grafana_config.vm.network "public_network", ip: "192.168.1.3", netmask: "255.255.255.0", bridge: "eth1"
    grafana_config.vm.hostname = "grafana"

    grafana_config.vm.provider "virtualbox" do |v|
      v.memory = 4096
      v.cpus = 2
    end

    grafana_config.vm.provision "shell", inline: <<-SHELL
      
      sudo apt update && sudo apt upgrade -y
      wget https://github.com/prometheus/prometheus/releases/download/v2.37.6/prometheus-2.37.6.linux-amd64.tar.gz
      tar xvfz prometheus-*.tar.gz
      rm prometheus-*.tar.gz
      sudo mkdir /etc/prometheus /var/lib/prometheus
      cd prometheus-2.37.6.linux-amd64
      sudo mv prometheus promtool /usr/local/bin/
      sudo mv prometheus.yml /etc/prometheus/prometheus.yml
      sudo mv consoles/ console_libraries/ /etc/prometheus/
      sudo useradd -rs /bin/false prometheus
      sudo chown -R prometheus: /etc/prometheus /var/lib/prometheus
      echo "[Unit]
      Description=Prometheus
      After=network.target

      [Service]
      ExecStart=/usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/var/lib/prometheus --web.console.templates=/etc/prometheus/consoles --web.console.libraries=/etc/prometheus/console_libraries --web.listen-address=0.0.0.0:9090
      ExecReload=/bin/kill -HUP $MAINPID
      KillSignal=SIGTERM
      User=prometheus
      Group=prometheus

      [Install]
      WantedBy=default.target" | sudo tee /etc/systemd/system/prometheus.service

      sudo systemctl daemon-reload
      sudo systemctl enable prometheus
      sudo systemctl start prometheus
      
      sudo apt-get install -y apt-transport-https software-properties-common

      sudo wget -q -O /usr/share/keyrings/grafana.key https://apt.grafana.com/gpg.key
      echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

      sudo apt-get update
      sudo apt-get install grafana -y

      sudo systemctl daemon-reload
      sudo systemctl enable grafana-server
      sudo systemctl start grafana-server
      
      
    SHELL
  end
end




