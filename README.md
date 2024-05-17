#Authentication (self-signed Certificate) between prometheus Server and target.

first we will start with :

Prometheus Server & Configuration:

Steps:

1-Configure the Prometheus server with a static IP address

First, update your package list to ensure you access the most recent versions available

                                          $ sudo apt update
2-Next, install the Prometheus package:

                  $ sudo apt install prometheus prometheus-node-exporter prometheus-pushgateway prometheus-alertmanager
3-Check the main configuration file for Prometheus is located at /etc/prometheus/prometheus.yml.

                       $ sudo vi /etc/prometheus/prometheus.yml
4-Enable Prometheus to start on boot and start the service

                             $ sudo systemctl enable prometheus

                              $ sudo systemctl start prometheus
5-You can also access Prometheus in a browser using the server’s IP address:

                                http://<PROMETHEUS_SERVER_IP>:9090
1.2 Prometheus Configuration: Steps:

1- Login to the server where Prometheus is running then edit the config file

                                 $ sudo vim /etc/prometheus/prometheus.yml
2- Locate the global.scrape_interval setting and change it to 10s:

global:

scrape_interval: 10s

3- Reload the Prometheus configuration:

                                        $ sudo systemctl restart Prometheus
2.1 Install & Configure Exporter:

1- Login to the machine that we will use for monitoring by Prometheus then create a user for Node Exporter:

                              $ sudo useradd -M -r -s /bin/false node_exporter
2- Download and extract the Node Exporter binary:

                 $ wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz


                           $ tar xvfz node_exporter-1.5.0.linux-amd64.tar.gz
3- Copy the Node Exporter binary to the appropriate location and set ownership:

                      $ sudo cp node_exporter-1.5.0.linux-amd64/node_exporter  /usr/local/bin/


                              $ sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
4- Create a systemd unit file for Node Exporter:

                                        $ sudo vim /etc/systemd/system/node_exporter.service
***** Define the Node Exporter service in the unit file:

                                 [Unit]

                                 Description=Prometheus Node Exporter

                                 Wants=network-online.target

                                  After=network-online.target


                                  [Service]

                                  User=node_exporter

                                   Group=node_exporter

                                    Type=simple

                                   ExecStart=/usr/local/bin/node_exporter


                                    [Install]

                                  WantedBy=multi-user.target
Start and enable the node_exporter service:

                                                  $ sudo systemctl daemon-reload

                                               $ sudo systemctl start node_exporter

                                            $ sudo systemctl enable node_exporter
2.2 You can retrieve the metrics served by Node Exporter like so:

                                         $ curl localhost:9100/metrics
3.1 Configure Prometheus to Scrape Metrics

1- Edit the Prometheus config file: $ sudo vim /etc/prometheus/prometheus.yml

******** Locate the scrape_configs section and add a new entry under that section. You will need to supply the private IP address of your new server for targets. ...

job_name: 'Linux Server'

static_configs:

targets: ['localhost:9100'] ...
Validate config file:

                                        $ promtool check config /etc/prometheus/prometheus.yml
Reload the Prometheus config:

                                             $ sudo systemctl restart Prometheus
Installing Grafana and Building dashboard
4.1 Login to the machine that we will use for monitoring by Prometheus

1- Install some required packages:

                                                           $ sudo apt-get update
                                       $ sudo apt-get install -y apt-transport-https software-properties-common wget
2- Add the GPG key for the Grafana OSS repository, then add the repository:

                                             $ wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
					 
                                                    $ sudo apt-get install ca-certificates
						
                                          $ sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
3- Install the Grafana package:

						  $ sudo apt-get update


                                                  $ sudo apt-get install grafana=9.2.3
4- Enable and start the grafana-server service: $ sudo systemctl enable grafana-server

                                                     $ sudo systemctl start grafana-server
5- Make sure the service is in the Active (running) state:

                                                    $ sudo systemctl status grafana-server
6- You can also verify that Grafana is working by accessing it in a web browser at:

                                                   http://<Grafana_Server_Public IP>:3000
7- Log in to Grafana with the username admin and password admin

   1- Reset the password when prompted.
   
   2- Click Add data source.
   
   3- Select Prometheus.
                                    For the URL, enter http://<Prometheus server Private IP>:9090.

   4- Click Save & Test. You should see a banner that says Data source is working.
   
   5- Test your setup by running a query to get some Prometheus data. Click the Explore icon on the left.
   
   6- In the PromQL Query input, enter a simple query, such as up.
   
   7- Execute the query. You should see some data appear.
# Authentication (self-signed Certificate) between prometheus Server and target.
1- Node Exporter Create a directory node_exporter mkdir ~/certs/

                                       cd  ~/certs/
2- generate certificates

                                $ openssl genrsa -out ~/certs/ca.key 2048

                                $ openssl req -x509 -new -nodes -key ~/certs/ca.key -sha256 -days 365 -out ~/certs/ca.crt
3- Create a web-config.yaml file which contains: tls_server_config:

                                cert_file: /etc/node_exporter/node_exporter.crt

                                key_file: /etc/node_exporter/node_exporter.key

                             basic_auth_users:

                                 prometheus: $2y$12$TAkyqJ8p1NkgERUyTf1y/OS3J/NIR4KK.2vFYODgpieHQVT8L.cw2
4- Update the service of node_exporter with TLS config

                                     [Unit] 
                                     Description=Node Exporter 
                                    Wants=network-online.target 
                                    After=network-online.target

                                     [Install]

                                     WantedBy=multi-user.target
5- Reload && start the node_exporter

                                          $ sudo systemctl daemon-reload

                                          $ sudo systemctl restart node_exporter
Now we have successfully created set-up for Authentication/Encryptio in Prometheus
