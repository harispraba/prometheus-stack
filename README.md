# Prometheus Stack Metrics Monitoring

Simple Prometheus Stack

## Create monitoring network

```bash
docker network create --driver bridge monitoring
```

## Create volumes

```bash
docker volume create --name alertmanager_data --driver local
docker volume create --name prometheus_data --driver local
docker volume create --name grafana_data --driver local
docker volume create --name grafana_logs --driver local
```

## Node Exporter Install

```bash
docker run -d --name node-exporter \
--restart unless-stopped \
--network monitoring \
-v /proc:/host/proc:ro \
-v /sys:/host/sys:ro \
-v /:/rootfs:ro \
prom/node-exporter:v1.8.0 \
--path.procfs=/host/proc \
--path.sysfs=/host/sys \
--path.rootfs=/rootfs \
--web.listen-address=:9100 \
--web.telemetry-path=/metrics
--collector.filesystem.mount-points-exclude="^/(dev|proc|sys|var/lib/docker/.+|var/lib/kubelet/.+)($|/)" \
--collector.textfile.directory /etc/node-exporter
```

## Alert Manager Install

```bash
docker run -d --name alertmanager \
--restart unless-stopped \
--network monitoring \
-v alertmanager_data:/alertmanager \
-v $(pwd)/alertmanager.yml:/etc/alertmanager/alertmanager.yml \
prom/alertmanager:v0.27.0 \
--config.file=/etc/alertmanager/alertmanager.yml \
--storage.path=/alertmanager \
--web.listen-address=:9093
```

## Prometheus Install

```bash
docker run -d --name prometheus \
--restart unless-stopped \
--network monitoring \
-v prometheus_data:/prometheus \
-v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
-v $(pwd)/alert_rules.yml:/etc/prometheus/alert_rules.yml \
prom/prometheus:v2.52.0 \
--config.file=/etc/prometheus/prometheus.yml \
--storage.tsdb.path=/prometheus \
--web.console.libraries=/etc/prometheus/console_libraries \
--web.console.templates=/etc/prometheus/consoles
```

## Grafana Install

```bash
docker run -d --name grafana \
--restart unless-stopped \
-p 3000:3000 \
--network monitoring \
-v grafana_data:/var/lib/grafana \
-v grafana_logs:/var/log/grafana \
-v $(pwd)/grafana.ini:/etc/grafana/grafana.ini \
-v $(pwd)/datasource.yml:/etc/grafana/provisioning/datasources/datasource.yml \
grafana/grafana:latest
```

### Grafana PromQL to Get CPU Usage

```sql
100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

### Grafana PromQL to Get Memory Usage

```sql
(node_memory_MemTotal_bytes - (node_memory_MemFree_bytes + node_memory_Buffers_bytes + node_memory_Cached_bytes)) / node_memory_MemTotal_bytes * 100
```
