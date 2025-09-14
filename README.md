# Open Virtual Network (OVN) Exporter

[![GitHub Release](https://img.shields.io/github/v/release/Liquescent-Development/ovn_exporter)](https://github.com/Liquescent-Development/ovn_exporter/releases)
[![Go Version](https://img.shields.io/badge/go-1.24-blue.svg)](https://go.dev/)
[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)

Export Open Virtual Network (OVN) data to Prometheus.

> **Note**: This is a modernized fork of the original [greenpau/ovn_exporter](https://github.com/greenpau/ovn_exporter) which is now archived. This version includes security improvements, updated dependencies, and compatibility with Go 1.24+.

## Features

This exporter collects metrics from the following OVN components:
* OVN `northd` service
* OVS `vswitchd` service
* `OVN Northbound` database
* `OVN Southbound` database
* `Open_vSwitch` database

## What's New in v2.0.0

### Breaking Changes
- **Go 1.24+ required** (upgraded from Go 1.20)
- **Module path changed** to `github.com/Liquescent-Development/ovn_exporter`
- **Logging system migrated** from deprecated `promlog` to `promslog/slog`
- **Metric names standardized** to follow Prometheus conventions:
  - `ovn_failed_req_count` → `ovn_failed_requests_total`
  - `ovn_log_file_size` → `ovn_log_file_size_bytes`
  - `ovn_db_file_size` → `ovn_db_file_size_bytes`
  - `ovn_log_event_count` → `ovn_log_events_total`
  - `ovn_memory_usage` → `ovn_memory_usage_bytes`
  - `ovn_next_poll` → `ovn_next_poll_timestamp_seconds`
- **Security improvements**:
  - Removed insecure `pprof` debug endpoints
  - Added HTTP server timeouts and security headers
  - Updated all dependencies to latest versions

### Improvements
- Multi-architecture support (Linux/Darwin on amd64/arm64)
- Automated releases via GitHub Actions
- All dependencies updated to latest versions
- Fixed deprecated Prometheus APIs

## Installation

### Pre-built Binaries

Download the latest release for your platform from the [releases page](https://github.com/Liquescent-Development/ovn_exporter/releases):

```bash
# Linux amd64
wget https://github.com/Liquescent-Development/ovn_exporter/releases/download/v2.0.0/ovn-exporter_2.0.0_linux_amd64.tar.gz
tar xvzf ovn-exporter_2.0.0_linux_amd64.tar.gz
sudo mv ovn-exporter /usr/local/bin/

# Linux arm64
wget https://github.com/Liquescent-Development/ovn_exporter/releases/download/v2.0.0/ovn-exporter_2.0.0_linux_arm64.tar.gz
tar xvzf ovn-exporter_2.0.0_linux_arm64.tar.gz
sudo mv ovn-exporter /usr/local/bin/
```

### Build from Source

```bash
git clone https://github.com/Liquescent-Development/ovn_exporter.git
cd ovn_exporter
make BUILD_OS=linux BUILD_ARCH=amd64
sudo make deploy BUILD_OS=linux BUILD_ARCH=amd64
```

### Systemd Service

Create a systemd service file `/etc/systemd/system/ovn-exporter.service`:

```ini
[Unit]
Description=OVN Exporter
After=network.target

[Service]
Type=simple
User=ovn-exporter
Group=openvswitch
ExecStart=/usr/local/bin/ovn-exporter
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

Then enable and start the service:

```bash
sudo useradd -r -s /bin/false ovn-exporter
sudo usermod -a -G openvswitch ovn-exporter
sudo systemctl daemon-reload
sudo systemctl enable ovn-exporter
sudo systemctl start ovn-exporter
```

## Configuration

The exporter can be configured via command-line flags. Run `ovn-exporter --help` to see all available options.

### Common Flags

| Flag | Default | Description |
|------|---------|-------------|
| `--web.listen-address` | `:9476` | Address to listen on for web interface and telemetry |
| `--web.telemetry-path` | `/metrics` | Path under which to expose metrics |
| `--ovn.timeout` | `2` | Timeout on gRPC requests to OVN (seconds) |
| `--ovn.poll-interval` | `15` | Minimum interval between collections from OVN server (seconds) |
| `--log.level` | `info` | Logging severity level (debug, info, warn, error) |
| `--version` | `false` | Show version information and exit |

### System Configuration

| Flag | Default | Description |
|------|---------|-------------|
| `--system.run.dir` | `/var/run/openvswitch` | OVS default run directory |

### OVS Database Configuration

| Flag | Default | Description |
|------|---------|-------------|
| `--database.vswitch.name` | `Open_vSwitch` | Name of OVS database |
| `--database.vswitch.socket.remote` | `unix:/var/run/openvswitch/db.sock` | JSON-RPC unix socket to OVS db |
| `--database.vswitch.file.data.path` | `/etc/openvswitch/conf.db` | OVS database file |
| `--database.vswitch.file.log.path` | `/var/log/openvswitch/ovsdb-server.log` | OVS database log file |
| `--database.vswitch.file.pid.path` | `/var/run/openvswitch/ovsdb-server.pid` | OVS database process ID file |
| `--database.vswitch.file.system.id.path` | `/etc/openvswitch/system-id.conf` | OVS system ID file |

### OVN Northbound Database Configuration

| Flag | Default | Description |
|------|---------|-------------|
| `--database.northbound.name` | `OVN_Northbound` | Name of OVN Northbound database |
| `--database.northbound.socket.remote` | `unix:/run/openvswitch/ovnnb_db.sock` | JSON-RPC unix socket to OVN NB db |
| `--database.northbound.socket.control` | `unix:/run/openvswitch/ovnnb_db.ctl` | JSON-RPC unix socket to OVN NB app |
| `--database.northbound.file.data.path` | `/var/lib/openvswitch/ovnnb_db.db` | OVN NB database file |
| `--database.northbound.file.log.path` | `/var/log/openvswitch/ovsdb-server-nb.log` | OVN NB database log file |
| `--database.northbound.file.pid.path` | `/run/openvswitch/ovnnb_db.pid` | OVN NB database process ID file |
| `--database.northbound.port.default` | `6641` | OVN NB database network socket port |
| `--database.northbound.port.ssl` | `6631` | OVN NB database network socket secure port |
| `--database.northbound.port.raft` | `6643` | OVN NB database network port for clustering (raft) |

### OVN Southbound Database Configuration

| Flag | Default | Description |
|------|---------|-------------|
| `--database.southbound.name` | `OVN_Southbound` | Name of OVN Southbound database |
| `--database.southbound.socket.remote` | `unix:/run/openvswitch/ovnsb_db.sock` | JSON-RPC unix socket to OVN SB db |
| `--database.southbound.socket.control` | `unix:/run/openvswitch/ovnsb_db.ctl` | JSON-RPC unix socket to OVN SB app |
| `--database.southbound.file.data.path` | `/var/lib/openvswitch/ovnsb_db.db` | OVN SB database file |
| `--database.southbound.file.log.path` | `/var/log/openvswitch/ovsdb-server-sb.log` | OVN SB database log file |
| `--database.southbound.file.pid.path` | `/run/openvswitch/ovnsb_db.pid` | OVN SB database process ID file |
| `--database.southbound.port.default` | `6642` | OVN SB database network socket port |
| `--database.southbound.port.ssl` | `6632` | OVN SB database network socket secure port |
| `--database.southbound.port.raft` | `6644` | OVN SB database network port for clustering (raft) |

### Service Configuration

| Flag | Default | Description |
|------|---------|-------------|
| `--service.vswitchd.file.log.path` | `/var/log/openvswitch/ovs-vswitchd.log` | OVS vswitchd daemon log file |
| `--service.vswitchd.file.pid.path` | `/var/run/openvswitch/ovs-vswitchd.pid` | OVS vswitchd daemon process ID file |
| `--service.ovn.northd.file.log.path` | `/var/log/openvswitch/ovn-northd.log` | OVN northd daemon log file |
| `--service.ovn.northd.file.pid.path` | `/run/openvswitch/ovn-northd.pid` | OVN northd daemon process ID file |

### Example Configuration

```bash
# Basic configuration
ovn-exporter \
  --web.listen-address=:9476 \
  --ovn.poll-interval=30 \
  --log.level=info

# Custom paths configuration
ovn-exporter \
  --web.listen-address=:9476 \
  --system.run.dir=/custom/run/openvswitch \
  --database.northbound.socket.remote=unix:/custom/run/openvswitch/ovnnb_db.sock \
  --database.southbound.socket.remote=unix:/custom/run/openvswitch/ovnsb_db.sock \
  --log.level=debug
```

## Metrics

The exporter provides comprehensive metrics about your OVN infrastructure. See [METRICS.md](METRICS.md) for a complete list of available metrics.

Example metrics:
- `ovn_up` - Whether OVN stack is up (1) or down (0)
- `ovn_info` - OVN stack information
- `ovn_cluster_*` - Cluster health and Raft consensus metrics
- `ovn_logical_switch_*` - Logical network topology metrics
- `ovn_chassis_info` - Chassis status and information

## Prometheus Configuration

Add the following to your `prometheus.yml`:

```yaml
scrape_configs:
  - job_name: 'ovn'
    static_configs:
      - targets: ['localhost:9476']
```

## Development

### Requirements
- Go 1.24+
- Access to OVN/OVS Unix sockets (typically requires root or openvswitch group membership)

### Building
```bash
# Build for current platform
make BUILD_OS=linux BUILD_ARCH=amd64

# Run tests
make test BUILD_OS=linux BUILD_ARCH=amd64

# Create test OVN environment
make ovn

# Clean up test environment
make del-ovn
```

### Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Security

This version includes several security improvements:
- HTTP server configured with proper timeouts
- Security headers (X-Content-Type-Options, X-Frame-Options, CSP)
- Removed debug/profiling endpoints
- All dependencies regularly updated

If you discover a security vulnerability, please email security@liquescent.dev

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

This is a fork of the original [ovn_exporter](https://github.com/greenpau/ovn_exporter) by Paul Greenberg (@greenpau). Thank you for creating and maintaining the original project!

## Support

For issues and questions:
- Open an issue on [GitHub](https://github.com/Liquescent-Development/ovn_exporter/issues)
- Check the [METRICS.md](METRICS.md) for metric documentation