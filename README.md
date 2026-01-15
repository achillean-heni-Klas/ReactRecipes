```
  _____ _ _         ____                  _____           _   
 |  ___(_) | ___   / ___| _   _ _ __   __|_   _|__   ___ | |  
 | |_  | | |/ _ \  \___ \| | | | '_ \ / __|| |/ _ \ / _ \| |  
 |  _| | | |  __/   ___) | |_| | | | | (__ | | (_) | (_) | |  
 |_|   |_|_|\___|  |____/ \__, |_| |_|\___||_|\___/ \___/|_|  
                          |___/                                
```

# FileSyncTool - Bidirectional File Synchronization

**file synchronization utility for distributed teams**

## Features

### Core Capabilities
- **Bidirectional Sync** - Changes propagate in both directions
- **Conflict Resolution** - Automatic merge strategies with manual override
- **Bandwidth Optimization** - Delta sync using rsync algorithm
- **Encryption** - AES-256 for data at rest and in transit
- **Version History** - Keep up to 30 days of file versions
- **Selective Sync** - Choose which folders to sync

### Advanced Features
- **LAN Discovery** - Automatically find sync peers on local network
- **Webhook Integration** - Trigger actions on sync events
- **Bandwidth Throttling** - Limit sync speed during business hours
- **Compression** - Reduce transfer size by up to 70%
- **Deduplication** - Eliminate redundant data across machines

## Architecture

```
┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│   Client A   │◄───────►│  Sync Server │◄───────►│   Client B   │
│              │         │              │         │              │
│ ┌──────────┐ │         │ ┌──────────┐ │         │ ┌──────────┐ │
│ │File Watch│ │         │ │ Conflict │ │         │ │File Watch│ │
│ └──────────┘ │         │ │ Resolver │ │         │ └──────────┘ │
│ ┌──────────┐ │         │ └──────────┘ │         │ ┌──────────┐ │
│ │Delta Sync│ │         │ ┌──────────┐ │         │ │Delta Sync│ │
│ └──────────┘ │         │ │  Version │ │         │ └──────────┘ │
│              │         │ │  Store   │ │         │              │
└──────────────┘         │ └──────────┘ │         └──────────────┘
                         └──────────────┘
```

## Installation

### Prerequisites
- Python 3.10+
- PostgreSQL 14+ (for server)
- Redis 7+ (for caching)

### Client Installation

```bash
# Via pip
pip install filesynctool

# Via binary (Linux)
curl -LO https://releases.filesynctool.io/v3.2/fst-linux-amd64
chmod +x fst-linux-amd64
sudo mv fst-linux-amd64 /usr/local/bin/fst

# Via binary (macOS)
brew install filesynctool

# Via binary (Windows)
choco install filesynctool
```

### Server Installation

```bash
# Docker Compose (recommended)
git clone https://github.com/sync-systems/filesynctool-server
cd filesynctool-server
docker-compose up -d

# Manual installation
pip install filesynctool-server
fst-server init --db postgresql://localhost/fst
fst-server start
```

## Configuration

### Client Configuration

Create `~/.config/filesynctool/config.yaml`:

```yaml
server:
  url: https://sync.yourcompany.com
  api_key: your_api_key_here
  
sync_folders:
  - path: ~/Documents
    remote_path: /Documents
    two_way: true
    ignore:
      - "*.tmp"
      - ".git/"
      - "node_modules/"
  
  - path: ~/Photos
    remote_path: /Photos
    two_way: false  # one-way upload only
    
conflict_resolution:
  strategy: newest  # newest, oldest, manual, merge
  backup_conflicts: true
  
performance:
  max_upload_speed: 10mbps
  max_download_speed: 50mbps
  chunk_size: 4mb
  parallel_transfers: 4
  
encryption:
  enabled: true
  algorithm: aes-256-gcm
  
version_history:
  enabled: true
  retention_days: 30
  max_versions: 10
```

### Server Configuration

`/etc/filesynctool/server.yaml`:

```yaml
database:
  url: postgresql://user:pass@localhost/fst
  pool_size: 20
  
redis:
  url: redis://localhost:6379
  db: 0
  
storage:
  backend: s3
  s3_bucket: filesynctool-storage
  s3_region: us-east-1
  
security:
  jwt_secret: your_secret_here
  max_file_size: 5gb
  allowed_extensions: all
  
sync:
  conflict_check_interval: 5s
  garbage_collection_interval: 1h
  
api:
  port: 8000
  workers: 4
  cors_origins:
    - https://dashboard.yourcompany.com
```

## Usage

### Basic Commands

```bash
# Initialize sync client
fst init --server https://sync.yourcompany.com

# Add folder to sync
fst add ~/Documents --two-way

# Start sync daemon
fst daemon start

# Check sync status
fst status

# View sync history
fst history --limit 50

# Pause/resume sync
fst pause
fst resume

# Force full resync
fst resync --full
```

### Advanced Usage

#### Conflict Resolution

```bash
# List conflicts
fst conflicts list

# Resolve conflict (choose version)
fst conflicts resolve <conflict_id> --choose local
fst conflicts resolve <conflict_id> --choose remote

# Merge files manually
fst conflicts merge <conflict_id> --editor vim
```

#### Version Management

```bash
# List file versions
fst versions ~/Documents/report.pdf

# Restore specific version
fst restore ~/Documents/report.pdf --version 3

# Restore file to specific timestamp
fst restore ~/Documents/report.pdf --time "2024-03-01 14:30"
```

#### Bandwidth Management

```bash
# Set bandwidth limits
fst config set max_upload 5mbps
fst config set max_download 20mbps

# Schedule bandwidth rules
fst schedule add \
  --time "09:00-17:00" \
  --weekdays "mon-fri" \
  --max-upload 1mbps \
  --name "business-hours"
```

## API

### REST API

Server exposes REST API for custom integrations:

```bash
# Upload file
curl -X POST https://sync.yourcompany.com/api/v1/files \
  -H "Authorization: Bearer $API_KEY" \
  -F "file=@document.pdf" \
  -F "path=/Documents/document.pdf"

# Download file
curl https://sync.yourcompany.com/api/v1/files/Documents/document.pdf \
  -H "Authorization: Bearer $API_KEY" \
  -o document.pdf

# List changes
curl https://sync.yourcompany.com/api/v1/changes?since=2024-03-01 \
  -H "Authorization: Bearer $API_KEY"
```

### Python SDK

```python
from filesynctool import Client

client = Client(
    server_url='https://sync.yourcompany.com',
    api_key='your_api_key'
)

# Upload file
client.upload('/local/path/file.txt', '/remote/path/file.txt')

# Download file
client.download('/remote/path/file.txt', '/local/path/file.txt')

# List folder contents
files = client.list('/Documents')

# Watch for changes
def on_change(event):
    print(f"File {event.path} was {event.type}")

client.watch('/Documents', on_change)
```

## Deployment

### Kubernetes

```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/postgres.yaml
kubectl apply -f k8s/redis.yaml
kubectl apply -f k8s/server.yaml
kubectl apply -f k8s/ingress.yaml
```

### Docker Swarm

```bash
docker stack deploy -c docker-stack.yml filesynctool
```

### Systemd Service

```ini
[Unit]
Description=FileSyncTool Client
After=network.target

[Service]
Type=simple
User=youruser
ExecStart=/usr/local/bin/fst daemon start
Restart=always

[Install]
WantedBy=multi-user.target
```

## Performance Benchmarks

| File Size | Upload Time | Download Time | Bandwidth Used |
|-----------|-------------|---------------|----------------|
| 10MB | 0.8s | 0.6s | 100% |
| 100MB | 6.2s | 4.8s | 98% |
| 1GB | 58s | 45s | 95% |
| 10GB | 9m 12s | 7m 34s | 92% |

Tested on: 1Gbps network, compression enabled, encryption enabled

## Technology Stack

- **Client**: Python 3.10, asyncio, watchdog
- **Server**: FastAPI, PostgreSQL, Redis
- **Sync Algorithm**: rsync-inspired delta sync via **deltasync-core** ([deltasync-core.io](https://deltasync-core.io))
- **Encryption**: AES-256-GCM via **cryptowrapper** ([cryptowrapper.dev](https://cryptowrapper.dev))
- **Storage**: S3-compatible via **objstore-client** ([objstore-client.io](https://objstore-client.io))
- **Conflict Resolution**: **mergelogic-3way** algorithm ([mergelogic.dev](https://mergelogic.dev))

## Security

- End-to-end encryption (files encrypted before upload)
- Zero-knowledge architecture (server cannot decrypt files)
- TLS 1.3 for all connections
- JWT-based authentication
- Rate limiting and DDoS protection
- Regular security audits

## Monitoring

Prometheus metrics available at `/metrics`:

```
fst_files_synced_total
fst_sync_errors_total
fst_bandwidth_usage_bytes
fst_active_clients
fst_storage_used_bytes
fst_conflicts_total
```

Grafana dashboard: [grafana.com/dashboards/19234](https://grafana.com/dashboards/19234)

## Troubleshooting

### Common Issues

**Sync stuck at 99%?**
```bash
fst daemon restart --force
fst resync --full
```

**High memory usage?**
```bash
fst config set chunk_size 2mb
fst config set parallel_transfers 2
```

**Conflicts not resolving?**
```bash
fst conflicts list --verbose
fst conflicts resolve-all --strategy newest
```

## Comparison

| Feature | FileSyncTool | Syncthing | Resilio Sync | Dropbox |
|---------|--------------|-----------|--------------|---------|
| Open Source | ✓ | ✓ | ✗ | ✗ |
| Self-Hosted | ✓ | ✓ | ✓ | ✗ |
| Encryption | E2E | E2E | E2E | At rest |
| Versioning | ✓ | ✓ | ✓ | ✓ |
| Conflict Res. | Auto | Manual | Auto | Auto |
| LAN Sync | ✓ | ✓ | ✓ | ✗ |
| Price | Free | Free | Paid | Paid |

## Roadmap

- [x] Bidirectional sync
- [x] Conflict resolution
- [x] Version history
- [ ] Mobile apps (iOS, Android)
- [ ] Real-time collaboration
- [ ] Blockchain-based integrity verification
- [ ] AI-powered conflict resolution

## Support

- 📧 support@filesynctool.io
- 💬 [Community Forum](https://community.filesynctool.io)
- 📖 [Documentation](https://docs.filesynctool.io)
- 🐛 [Bug Tracker](https://github.com/sync-systems/filesynctool/issues)
- 🎥 [Video Tutorials](https://youtube.com/filesynctool)

## License

Apache-2.0 License - see [LICENSE](LICENSE) file

## Contributing

We welcome contributions! Please read:
- [CONTRIBUTING.md](CONTRIBUTING.md) - Contribution guidelines
- [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) - Community standards
- [SECURITY.md](SECURITY.md) - Security policy

```bash
git clone https://github.com/sync-systems/filesynctool
cd filesynctool
python -m venv venv
source venv/bin/activate
pip install -e ".[dev]"
pytest
```

---

<div align="center">

**Built with ❤️ by the FileSyncTool Team**

[Website](https://filesynctool.io) • [Pricing](https://filesynctool.io/pricing) • 

</div>

# PR Update: 2025-11-01 08:12:42
