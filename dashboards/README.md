# OVN Exporter Grafana Dashboards

This directory contains Grafana dashboard configurations designed for monitoring Open Virtual Network (OVN) infrastructure using metrics from the OVN exporter.

## Available Dashboards

### 1. OVN System Overview (`ovn-system-overview.json`)

**Purpose**: High-level infrastructure monitoring for system operators

**Key Features**:
- Overall OVN stack health status
- Real-time system availability metrics
- Infrastructure component distribution (chassis, switches, routers)
- Database and log file size monitoring
- Log event analysis by severity
- Chassis information table

**Target Audience**: System operators, infrastructure teams, NOC engineers

**Refresh Rate**: 30 seconds

### 2. OVN SLA/SLO Performance (`ovn-sla-performance.json`)

**Purpose**: Customer experience monitoring and service level objective tracking

**Key Features**:
- SLA uptime compliance gauges (configurable time periods)
- Network availability percentage with thresholds
- Error rate monitoring with SLA impact
- Response time tracking
- Active resource monitoring (chassis, tunnels)
- SLA compliance trends over time
- Performance impact analysis
- Memory usage by component

**Target Audience**: Service delivery teams, customer success managers, SLA compliance officers

**Refresh Rate**: 30 seconds

**SLA Thresholds**:
- **Green**: ≥99.9% uptime (excellent)
- **Yellow**: 99.0-99.9% uptime (warning)
- **Red**: <99.0% uptime (critical)

## Installation

### Import to Grafana

1. **Via Grafana UI**:
   - Navigate to Dashboards > Import
   - Upload the JSON file or paste the content
   - Configure your Prometheus datasource
   - Save the dashboard

2. **Via Provisioning** (recommended for production):
   ```yaml
   # dashboards.yml
   apiVersion: 1
   providers:
   - name: 'ovn-dashboards'
     orgId: 1
     folder: 'OVN Monitoring'
     type: file
     disableDeletion: false
     updateIntervalSeconds: 10
     options:
       path: /etc/grafana/provisioning/dashboards/ovn
   ```

3. **Using Grafana API**:
   ```bash
   curl -X POST \
     http://grafana:3000/api/dashboards/db \
     -H 'Content-Type: application/json' \
     -H 'Authorization: Bearer YOUR_API_KEY' \
     -d @ovn-system-overview.json
   ```

### Prerequisites

- Grafana 8.0+ (tested with 10.0)
- Prometheus datasource configured
- OVN Exporter running and scraping metrics

## Configuration

### Datasource Variables

Both dashboards use templated datasource variables:
- `${datasource}`: Select your Prometheus datasource

### Dashboard Variables

**OVN SLA/SLO Performance** includes:
- `${range}`: SLA measurement period (1h, 6h, 24h, 7d, 30d)

### Customization

#### Adjusting SLA Thresholds

Edit the gauge panels in `ovn-sla-performance.json`:

```json
"thresholds": {
  "steps": [
    {"color": "red", "value": null},
    {"color": "yellow", "value": 99.0},
    {"color": "green", "value": 99.9}
  ]
}
```

#### Adding Custom Metrics

Add new panels using available OVN metrics:
- `ovn_up` - OVN stack status
- `ovn_failed_requests_total` - Request failures
- `ovn_chassis_info` - Chassis information
- `ovn_logical_switch_*` - Logical switch metrics
- `ovn_cluster_*` - Cluster status metrics

See [METRICS.md](../METRICS.md) for complete metric reference.

## Alerting Integration

### Recommended Alerts

1. **OVN Stack Down**:
   ```promql
   ovn_up == 0
   ```

2. **High Error Rate**:
   ```promql
   rate(ovn_failed_requests_total[5m]) > 0.01
   ```

3. **SLA Breach**:
   ```promql
   (
     (increase(ovn_up[1h]) - increase(ovn_failed_requests_total[1h])) /
     increase(ovn_up[1h])
   ) * 100 < 99.0
   ```

4. **Chassis Offline**:
   ```promql
   ovn_chassis_info == 0
   ```

### Alert Manager Integration

Configure alerts to integrate with your notification systems (Slack, PagerDuty, email):

```yaml
# alertmanager.yml
route:
  group_by: ['alertname', 'cluster', 'service']
  routes:
  - match:
      severity: critical
    receiver: 'pagerduty'
  - match:
      severity: warning
    receiver: 'slack'

receivers:
- name: 'pagerduty'
  pagerduty_configs:
  - service_key: 'YOUR_PAGERDUTY_KEY'
```

## Troubleshooting

### Common Issues

1. **No Data Displayed**:
   - Verify OVN Exporter is running and accessible
   - Check Prometheus is scraping the exporter
   - Confirm datasource configuration in Grafana

2. **Metrics Missing**:
   - Ensure OVN services are running
   - Check exporter has access to OVN sockets
   - Verify metric names match your OVN Exporter version

3. **Performance Issues**:
   - Adjust refresh rates for large deployments
   - Consider using recording rules for complex queries
   - Optimize time ranges for historical data

### Validation Queries

Test your setup with these basic queries:

```promql
# Basic connectivity
ovn_up

# Request rate
rate(ovn_failed_requests_total[5m])

# Chassis count
count(ovn_chassis_info)

# Logical switch count
count(count by (name) (ovn_logical_switch_info))
```

## Support

For dashboard-related issues:
- Check the [main repository](https://github.com/Liquescent-Development/ovn_exporter)
- Review [METRICS.md](../METRICS.md) for metric definitions
- Open an issue with dashboard JSON and error details