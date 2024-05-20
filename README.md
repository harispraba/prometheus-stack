# Prometheus Stack Metrics Monitoring

Simple Prometheus Stack

## Create monitoring network

```bash
docker network create monitoring
```

## Node Exporter Install

```bash
docker run \
-it \
-d \
--network monitoring \
-p 9100:9100 \
--name node-exporter \
-v /proc:/host/proc \
-v /sys:/host/sys \
-v /:/rootfs \
-v /etc/hostname:/etc/hostname \
prom/node-exporter:v1.8.0 \
--path.procfs /host/proc \
--path.sysfs /host/sys \
--collector.filesystem.mount-points-exclude=”^/(dev|proc|sys|var/lib/docker/.+|var/lib/kubelet/.+)($|/)” \
--collector.textfile.directory /etc/node-exporter
```

## Alert Manager Install

```bash
docker run \
--network monitoring \
--name alertmanager \
-d -p 9093:9093 \
-v ./alertmanager.yml:/etc/alertmanager/alertmanager.yml \
prom/alertmanager:v0.27.0
```

## Prometheus Install

```bash
docker run \
--network monitoring \
--name prometheus-server \
-d \
-p 9090:9090 \
-v “./prometheus.yml:/etc/prometheus/prometheus.yml” \
-v “./alert_rules.yml:/etc/prometheus/alert_rules.yml” \
prom/prometheus:v2.52.0
```

## Grafana Install

```bash
docker run \
--network monitoring \
--name grafana \
-d \
-p 3000:3000 \
-v ./grafana.ini:/etc/grafana/grafana.ini \
grafana/grafana:latest
```
