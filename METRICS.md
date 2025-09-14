# OVN Exporter Metrics

This document provides a comprehensive list of all metrics exported by the OVN Exporter.

## Metric Naming Conventions

This exporter follows [Prometheus metric naming best practices](https://prometheus.io/docs/practices/naming/):
- All metrics are prefixed with `ovn_`
- Metrics use base units (seconds, bytes, etc.)
- Counter metrics end with `_total`
- Units are included in metric names where appropriate (`_bytes`, `_seconds`)
- Info metrics end with `_info`

> **Note**: Some legacy metric names are retained for backward compatibility but may be deprecated in future versions.

## Core Metrics

### System Status

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `ovn_up` | Gauge | Whether OVN stack is up (1) or down (0) | - |
| `ovn_info` | Gauge | Basic information about OVN stack (always 1) | `system_id`, `rundir`, `hostname`, `system_type`, `system_version`, `ovs_version`, `db_version` |
| `ovn_exporter_build_info` | Gauge | Build information about the exporter (always 1) | `version`, `revision`, `branch`, `goversion` |

### Request and Polling Metrics

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `ovn_failed_requests_total` | Counter | Total number of failed requests to OVN stack | `system_id` |
| `ovn_next_poll_timestamp_seconds` | Counter | Timestamp of the next potential poll of OVN stack (Unix timestamp) | `system_id` |

### Process Metrics

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `ovn_pid` | Gauge | Process ID of a running OVN component (0 if not running) | `system_id`, `component`, `user`, `group` |

### File Size Metrics

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `ovn_log_file_size_bytes` | Gauge | Size of a log file associated with an OVN component in bytes | `system_id`, `component`, `filename` |
| `ovn_db_file_size_bytes` | Gauge | Size of a database file associated with an OVN component in bytes | `system_id`, `component`, `filename` |

### Log Event Metrics

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `ovn_log_events_total` | Counter | Total number of recorded log messages by severity and source | `system_id`, `component`, `severity`, `source` |

## Cluster Metrics

### Cluster Status

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `ovn_cluster_enabled` | Gauge | Whether OVN clustering is enabled (1) or not (0) | `system_id`, `component` |
| `ovn_cluster_status` | Gauge | Status of server in cluster (1=member, 0=other) | `system_id`, `component`, `server_id`, `cluster_id` |
| `ovn_cluster_role` | Gauge | Role in cluster (3=leader, 2=candidate, 1=follower, 0=other) | `system_id`, `component`, `server_id`, `cluster_id` |
| `ovn_cluster_group` | Gauge | Cluster group participation (always 1) | `system_id`, `cluster_group` |

### Cluster Peer Connections

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `ovn_cluster_inbound_peer_connected` | Gauge | Cluster peer connected to this server (always 1 when connected) | `system_id`, `component`, `server_id`, `cluster_id`, `peer_id`, `peer_address` |
| `ovn_cluster_outbound_peer_connected` | Gauge | This server connected to cluster peer (always 1 when connected) | `system_id`, `component`, `server_id`, `cluster_id`, `peer_id`, `peer_address` |
| `ovn_cluster_inbound_peer_conn_total` | Gauge | Total number of inbound connections from cluster peers | `system_id`, `component`, `server_id`, `cluster_id` |
| `ovn_cluster_outbound_peer_conn_total` | Gauge | Total number of outbound connections to cluster peers | `system_id`, `component`, `server_id`, `cluster_id` |
| `ovn_cluster_peer_count` | Gauge | Total number of peers in this server's cluster | `system_id`, `component`, `server_id`, `cluster_id` |

### Raft Consensus Metrics

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `ovn_cluster_term` | Counter | Current raft term known by this server | `system_id`, `component`, `server_id`, `cluster_id` |
| `ovn_cluster_leader_self` | Gauge | Whether this server considers itself a leader (1) or not (0) | `system_id`, `component`, `server_id`, `cluster_id` |
| `ovn_cluster_vote_self` | Gauge | Whether this server voted itself as leader (1) or not (0) | `system_id`, `component`, `server_id`, `cluster_id` |

### Raft Log Metrics

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `ovn_cluster_log_low_index` | Counter | Raft log low index for this server | `system_id`, `component`, `server_id`, `cluster_id` |
| `ovn_cluster_log_high_index` | Counter | Raft log high index for this server | `system_id`, `component`, `server_id`, `cluster_id` |
| `ovn_cluster_next_index` | Counter | Raft next index for this server | `system_id`, `component`, `server_id`, `cluster_id` |
| `ovn_cluster_match_index` | Counter | Raft match index for this server | `system_id`, `component`, `server_id`, `cluster_id` |
| `ovn_cluster_uncommitted_entry_count` | Gauge | Number of raft entries not yet committed | `system_id`, `component`, `server_id`, `cluster_id` |
| `ovn_cluster_pending_entry_count` | Gauge | Number of raft entries not yet applied | `system_id`, `component`, `server_id`, `cluster_id` |

### Raft Peer Metrics

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `ovn_cluster_peer_next_index` | Counter | Raft next index for cluster peer | `system_id`, `component`, `server_id`, `cluster_id`, `peer_id` |
| `ovn_cluster_peer_match_index` | Counter | Raft match index for cluster peer | `system_id`, `component`, `server_id`, `cluster_id`, `peer_id` |

## Network Metrics

### Chassis Metrics

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `ovn_chassis_info` | Gauge | Chassis status (1=up, 0=down) with additional info | `system_id`, `uuid`, `name`, `ip` |

### Network Ports

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `ovn_network_port` | Gauge | TCP port used for database connection (0 if not in use) | `system_id`, `component`, `usage` |

## Logical Network Metrics

### Logical Switches

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `ovn_logical_switch_info` | Gauge | Information about logical switch (always 1) | `system_id`, `uuid`, `name` |
| `ovn_logical_switch_external_id` | Gauge | External IDs for logical switches (always 1) | `system_id`, `uuid`, `key`, `value` |
| `ovn_logical_switch_tunnel_key` | Gauge | Tunnel key value for logical switch | `system_id`, `uuid` |
| `ovn_logical_switch_ports` | Gauge | Number of ports connected to logical switch | `system_id`, `uuid` |

### Logical Switch Ports

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `ovn_logical_switch_port_info` | Gauge | Information about logical switch port (always 1) | `system_id`, `uuid`, `name`, `chassis`, `datapath`, `port_binding`, `mac_address`, `ip_address` |
| `ovn_logical_switch_port_binding` | Gauge | Association between logical switch and port (always 1) | `system_id`, `uuid`, `port` |
| `ovn_logical_switch_port_tunnel_key` | Gauge | Tunnel key value for logical switch port | `system_id`, `uuid` |

## Performance Metrics

### Coverage Metrics

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `ovn_coverage_avg` | Gauge | Average rate of events during OVSDB daemon runtime | `system_id`, `component`, `event`, `interval` |
| `ovn_coverage_total` | Counter | Total count of events during OVSDB daemon runtime | `system_id`, `component`, `event` |

### Memory Usage

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `ovn_memory_usage_bytes` | Gauge | Memory usage in bytes | `system_id`, `component`, `facility` |

## Label Descriptions

### Common Labels

- `system_id`: Unique identifier for the OVN system
- `component`: OVN component (e.g., `ovsdb-server-northbound`, `ovsdb-server-southbound`, `ovn-northd`, `ovs-vswitchd`)
- `uuid`: Universally unique identifier for the resource

### Cluster Labels

- `server_id`: Raft server ID in the cluster
- `cluster_id`: Raft cluster ID
- `peer_id`: ID of a cluster peer
- `peer_address`: Network address of a cluster peer
- `cluster_group`: Combined cluster group identifier

### Network Labels

- `chassis`: Chassis name or identifier
- `datapath`: Datapath identifier
- `port_binding`: Port binding identifier
- `mac_address`: MAC address
- `ip_address`: IP address
- `usage`: Port usage type (e.g., `default`, `ssl`, `raft`)

### Event Labels

- `event`: Event name (for coverage metrics)
- `interval`: Time interval for averaged metrics (e.g., `5s`, `5m`, `1h`)
- `severity`: Log severity level
- `source`: Log source

## Prometheus Query Examples

### Check OVN Health
```promql
# OVN stack status
ovn_up

# Cluster leader status
ovn_cluster_role{cluster_id=~".+"} == 3
```

### Monitor Cluster Health
```promql
# Number of cluster members
sum by (cluster_id) (ovn_cluster_status == 1)

# Uncommitted entries (potential replication lag)
ovn_cluster_uncommitted_entry_count > 0
```

### Track Logical Network Size
```promql
# Total logical switches
count(ovn_logical_switch_info)

# Total logical switch ports
sum(ovn_logical_switch_ports)
```

### Performance Monitoring
```promql
# Memory usage by component
ovn_memory_usage_bytes

# Failed request rate
rate(ovn_failed_requests_total[5m])
```

## Metric Naming Changes in v2.0.0

The following metrics were renamed in v2.0.0 to conform to Prometheus naming conventions:

| Old Name (v1.x) | New Name (v2.x) | Type |
|-----------------|------------------|------|
| `ovn_failed_req_count` | `ovn_failed_requests_total` | Counter |
| `ovn_log_file_size` | `ovn_log_file_size_bytes` | Gauge |
| `ovn_db_file_size` | `ovn_db_file_size_bytes` | Gauge |
| `ovn_log_event_count` | `ovn_log_events_total` | Counter |
| `ovn_memory_usage` | `ovn_memory_usage_bytes` | Gauge |
| `ovn_next_poll` | `ovn_next_poll_timestamp_seconds` | Counter |