
# Cisco Telemetry TIG Stack

Repository holding the files to build the Telemetry Stack (TIG) for NetDevOps Webinar

This docker image contains the following versions:

* Docker Image:      2.3.0
* Ubuntu:            18.04
* InfluxDB:          1.7.10
* Telegraf (StatsD): 1.13.3-1
* Grafana:           6.6.2

This Docker image was based upon the work of Samuele Bistoletti
https://github.com/samuelebistoletti/docker-statsd-influxdb-grafana

## Quick Start

First, create your own telegraf.pem and telegraf.key using the following commands:

```
openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -extensions v3_ca -out rootCA.pem 
openssl genrsa -out telegraf.key 2048
openssl req -new -key telegraf.key -out telegraf.csr 
openssl x509 -req -in telegraf.csr -CA rootCA.pem -CAkey telegraf.key -CAcreateserial -out telegraf.pem -days 500 -sha256 
```
Replace the files in telegraf/ with the newly generated files.

To build the image:

1) No TLS:
```sh
docker image build -f Dockerfile-notls -t <your id>/<your name of image>:no-tls
```

2) TLS:
```sh
docker image build -f Dockerfile-tls -t <your id>/<your name of image>:tls
```

To start the container the first time launch:

No TLS
```sh
docker run --ulimit nofile=66000:66000 \
  -d \
  --name docker-statsd-influxdb-grafana \
  -p 3003:3003 \
  -p 3004:8888 \
  -p 8086:8086 \
  -p <NCS1004 GRPC port>:<NCS1004 GRPC port> \
  <your id>/<name_of_image>:no-tls
```

TLS
```sh
docker run --ulimit nofile=66000:66000 \
  -d \
  --name docker-statsd-influxdb-grafana \
  -p 3003:3003 \
  -p 3004:8888 \
  -p 8086:8086 \
  -p <NCS1004 GRPC port>:<NCS1004 GRPC port> \
  <your id>/<name_of_image>:tls
```

Finally, copy the pem file to the XR device from bash, like this (destination file has to be dialout.pem in that path):

``` 
RP/0/RP0/CPU0:NCS1004_70#bashThu Jun 25 15:45:53.851 UTC[NCS1004_70:~]$ scp root@10.201.213.12:/root/telegraf.pem /misc/config/grpc/dialout/dialout.pem
root@10.201.213.12's password:telegraf.pem                                 100% 1220     1.2KB/s   00:00 
``` 
## Mapped Ports

```
Host		Container		Service

3003		3003			grafana
3004		8888			influxdb-admin (chronograf)
8086		8086			influxdb
8125		8125			statsd
```

## Grafana

Open <http://localhost:3003>

```
Username: root
Password: root
```

### Add data source on Grafana

1. Using the wizard click on `Add data source`
2. Choose a `name` for the source and flag it as `Default`
3. Choose `InfluxDB` as `type`
4. Choose `direct` as `access`
5. Fill remaining fields as follows and click on `Add` without altering other fields

```
Url: http://localhost:8086
Database:	telegraf
User: telegraf
Password:	telegraf
```

Basic auth and credentials must be left unflagged. Proxy is not required.

Now you are ready to add your first dashboard and launch some query on database.

## InfluxDB

### Web Interface

Open <http://localhost:3004>

```
Username: root
Password: root
Port: 8086
```

### InfluxDB Shell (CLI)

1. Attach to docker container, run shell `/bin/bash`
2. Launch `influx` to open InfluxDB Shell (CLI)
