# Zabbix Redpanda Monitoring Template

<div align="center">

![Zabbix](https://img.shields.io/badge/Zabbix-7.0-red?style=for-the-badge&logo=zabbix&logoColor=white)
![Redpanda](https://img.shields.io/badge/Redpanda-25.x-E63946?style=for-the-badge&logo=apache-kafka&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

**Complete Redpanda cluster monitoring with automatic broker, topic, and connector discovery**

</div>

---

## 📋 Overview

A comprehensive monitoring solution for Redpanda clusters using Zabbix HTTP Agent. Collects metrics from both Prometheus endpoint (`/public_metrics`) and Admin API (`/v1/cluster_view`), providing full visibility into cluster health, brokers, topics, and Kafka Connect connectors.

### ✨ Features

- **Cluster Monitoring**: Total brokers, online status, disk usage, CPU cores
- **Broker Discovery**: Auto-discovers all brokers with individual health metrics
- **Topic Discovery**: Auto-discovers all topics with partition, replica, and offset metrics
- **Connector Discovery**: Auto-discovers Kafka Connect connectors and tasks
- **Performance Metrics**: Records produced/fetched, RPC connections, I/O operations
- **Health Alerts**: Triggers for offline brokers, under-replicated partitions, disk usage

## 📦 Requirements

| Component | Version |
|-----------|---------|
| Zabbix Server | 7.0+ |
| Redpanda | 23.x+ |
| Admin API | Port 9644 (default) |
| Connect API | Port 8083 (optional) |

## 🚀 Quick Start

### 1. Import Template

1. Go to **Data collection** → **Templates**
2. Click **Import**
3. Select `zbx_redpanda_template.yaml`
4. Click **Import**

### 2. Configure Host

1. Go to **Data collection** → **Hosts**
2. Create a new host or select existing
3. Link template `Redpanda by HTTP`
4. Configure macros:

| Macro | Value | Description |
|-------|-------|-------------|
| `{$REDPANDA.HOST}` | `10.0.0.1` | Redpanda broker IP/DNS |
| `{$REDPANDA.PROTOCOL}` | `http` | Protocol (http/https) |
| `{$REDPANDA.PORT}` | `8083` | Kafka Connect port |

### 3. Verify Data Collection

After ~2 minutes, check:
- **Monitoring** → **Latest data** → Filter by host
- **Monitoring** → **Discovery** → Check discovered brokers/topics

## 📊 Collected Metrics

### Cluster Metrics (Admin API)

| Metric | Key | Description |
|--------|-----|-------------|
| Total Brokers | `redpanda.cluster.brokers.total` | Number of brokers in cluster |
| Brokers Online | `redpanda.cluster.brokers.online` | Brokers that are alive |
| Brokers in Maintenance | `redpanda.cluster.brokers.maintenance` | Brokers draining |
| Brokers in Recovery | `redpanda.cluster.brokers.recovery` | Brokers in recovery mode |
| Total Cores | `redpanda.cluster.total.cores` | CPU cores across cluster |
| Cluster Disk Free | `redpanda.cluster.disk.free` | Total free disk space |
| Cluster Disk Total | `redpanda.cluster.disk.total` | Total disk capacity |
| Cluster Disk Used % | `redpanda.cluster.disk.pused` | Disk usage percentage |

### Application Metrics (Prometheus)

| Metric | Key | Description |
|--------|-----|-------------|
| Uptime | `redpanda.application.uptime` | Application uptime in seconds |
| Version | `redpanda.application.version` | Redpanda version |
| Revision | `redpanda.application.revision` | Git revision |
| FIPS Mode | `redpanda.application.fips` | FIPS mode enabled (0/1) |
| License Expiry | `redpanda.cluster.license.expiry` | Seconds until license expires |

### Storage Metrics

| Metric | Key | Description |
|--------|-----|-------------|
| Disk Total | `redpanda.storage.disk.total` | Total disk bytes |
| Disk Free | `redpanda.storage.disk.free` | Free disk bytes |
| Disk Used % | `redpanda.storage.disk.pused` | Disk usage percentage |
| Disk Alert | `redpanda.storage.disk.alert` | Disk space alert (0/1/2) |

### Kafka Metrics

| Metric | Key | Description |
|--------|-----|-------------|
| Total Partitions | `redpanda.kafka.partitions.total` | Total partitions |
| Total Replicas | `redpanda.kafka.replicas.total` | Total replicas |
| Under-replicated | `redpanda.kafka.under.replicated` | Under-replicated replicas |
| Records Produced | `redpanda.kafka.records.produced` | Total records produced |
| Records Fetched | `redpanda.kafka.records.fetched` | Total records fetched |
| Request Bytes | `redpanda.kafka.request.bytes` | Total request bytes |
| Consumer Groups | `redpanda.kafka.consumer.groups` | Number of consumer groups |

### RPC Metrics

| Metric | Key | Description |
|--------|-----|-------------|
| Kafka Connections | `redpanda.rpc.connections.kafka` | Active Kafka connections |
| Internal Connections | `redpanda.rpc.connections.internal` | Active internal connections |
| Kafka Errors | `redpanda.rpc.errors.kafka` | Kafka RPC errors |
| Internal Errors | `redpanda.rpc.errors.internal` | Internal RPC errors |

### Raft Metrics

| Metric | Key | Description |
|--------|-----|-------------|
| Leadership Changes | `redpanda.raft.leadership.changes` | Total leadership changes |
| Partitions to Recover | `redpanda.raft.recovery.partitions` | Partitions pending recovery |
| Active Recovery | `redpanda.raft.recovery.active` | Partitions actively recovering |

## 🔍 Discovery Rules

### Broker Discovery

Discovers all brokers in the cluster via Admin API.

| Macro | Description |
|-------|-------------|
| `{#NODE_ID}` | Broker node ID |
| `{#BROKER_ADDRESS}` | Broker hostname |
| `{#BROKER_VERSION}` | Redpanda version |

**Per-Broker Items:**

| Item | Description |
|------|-------------|
| Is Alive | Broker online status (0/1) |
| Membership Status | active/draining/removed |
| Maintenance Draining | In maintenance mode (0/1) |
| Recovery Mode | In recovery mode (0/1) |
| Disk Free | Free disk space |
| Disk Total | Total disk space |
| Disk Used % | Disk usage percentage |
| Version | Redpanda version |
| Num Cores | CPU cores |

### Topic Discovery

Discovers all topics via Prometheus metrics.

| Macro | Description |
|-------|-------------|
| `{#TOPIC}` | Topic name |

**Per-Topic Items:**

| Item | Description |
|------|-------------|
| Partitions | Number of partitions |
| Replicas | Number of replicas |
| Max Offset | High watermark (sum) |
| Under-replicated | Under-replicated replicas |
| Records Produced | Total records produced |
| Records Fetched | Total records fetched |
| Leadership Changes | Raft leadership changes |

**Filter:** Excludes internal topics starting with `_`, `controller`, `id_allocator`.

### Connector Discovery

Discovers Kafka Connect connectors via Connect API.

| Macro | Description |
|-------|-------------|
| `{#CONNECTORS}` | Connector name |
| `{#TASKS.ID}` | Task ID |

**Per-Connector Items:**

| Item | Description |
|------|-------------|
| Connector Status | RUNNING/PAUSED/FAILED |
| Task Status | Task state per task ID |

## 🚨 Triggers

### Disaster

| Trigger | Description |
|---------|-------------|
| Broker is OFFLINE | A broker is not alive |
| Connector STOPPED | Kafka Connect connector not running |
| Connector Task STOPPED | Connector task not running |

### High

| Trigger | Description |
|---------|-------------|
| Some brokers offline | `brokers.online < brokers.total` |
| Cluster disk critical | Disk usage > 90% |
| Broker disk critical | Per-broker disk > 90% |
| Storage disk degraded | Disk space alert level 2 |

### Warning

| Trigger | Description |
|---------|-------------|
| License expires < 7 days | Enterprise license expiring |
| Partitions broken rack | Rack constraint violated |
| Brokers in recovery | Brokers in recovery mode |
| Broker not active | Membership not "active" |
| Cluster disk high | Disk usage > 80% |
| Broker disk high | Per-broker disk > 80% |
| Under-replicated partitions | Global under-replication |
| Topic under-replicated | Per-topic under-replication |
| Storage disk low | Disk space alert level 1 |
| Node RPCs timing out | More than 5 timeouts |

### Info

| Trigger | Description |
|---------|-------------|
| Version changed | Redpanda version updated |
| Broker in maintenance | Broker draining for maintenance |
| Partitions recovering | Partitions in recovery |

## ⚙️ Macros

| Macro | Default | Description |
|-------|---------|-------------|
| `{$REDPANDA.HOST}` | - | Redpanda broker IP/DNS |
| `{$REDPANDA.PROTOCOL}` | `http` | Protocol (http/https) |
| `{$REDPANDA.PORT}` | `8083` | Kafka Connect API port |
| `{$REDPANDA.DISK.WARN.PUSED}` | `80` | Disk warning threshold % |
| `{$REDPANDA.DISK.CRIT.PUSED}` | `90` | Disk critical threshold % |

## 📁 Files

```
├── README.md                      # This documentation
|── 7.0/
|   ├── zbx_redpanda_template.yaml # Zabbix 7.0 template
```

## 🔌 API Endpoints Used

| Endpoint | Port | Description |
|----------|------|-------------|
| `/public_metrics` | 9644 | Prometheus metrics |
| `/v1/cluster_view` | 9644 | Cluster and broker info |
| `/connectors` | 8083 | Kafka Connect connectors |
| `/connectors/{name}/status` | 8083 | Connector status |

## 🐛 Troubleshooting

### Test Prometheus Metrics

```bash
curl -s "http://REDPANDA_HOST:9644/public_metrics"
```

### Test Admin API

```bash
curl -s "http://REDPANDA_HOST:9644/v1/cluster_view" | jq .
```

### Test Connect API

```bash
curl -s "http://REDPANDA_HOST:8083/connectors" | jq .
```

### Common Issues

| Issue | Solution |
|-------|----------|
| No data collected | Verify firewall allows 9644 and 8083 |
| Connection refused | Check Redpanda is running |
| Empty metrics | Verify Prometheus endpoint is enabled |
| Discovery not working | Wait for discovery interval (1h default) |
| Preprocessing errors | Check JSONPath syntax in template |

## 📈 Dashboard Recommendations

Create dashboards with:

1. **Cluster Overview**
   - Total brokers vs online
   - Cluster disk usage gauge
   - Total partitions/replicas

2. **Broker Health**
   - Per-broker disk usage
   - Broker status map
   - Maintenance status

3. **Topic Performance**
   - Records produced/fetched graphs
   - Under-replicated topics
   - Top topics by offset

4. **Connector Status**
   - Connector/task status
   - Error counters

## 👤 Author

**vfcastr** - [@vfcastr](https://github.com/vfcastr)

## 📄 License

MIT License
