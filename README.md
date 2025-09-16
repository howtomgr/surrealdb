# SurrealDB Installation Guide

SurrealDB is a free and open-source multi-model database designed for modern applications. It combines the functionality of a traditional database, a document store, a graph database, and a real-time collaborative backend. SurrealDB serves as a FOSS alternative to proprietary multi-model databases like FaunaDB, Amazon DynamoDB, or Google Firestore, offering SQL-style query language, real-time subscriptions, and advanced permissions in a single scalable platform.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

### Hardware Requirements
- **CPU**: Modern 64-bit processor (x86_64 or ARM64)
- **RAM**: 1GB minimum (4GB+ recommended for production)
- **Storage**: 10GB+ for database files
- **Network**: Low latency for real-time features

### Software Requirements
- **Operating System**: Linux, macOS, or Windows
- **Optional**: Docker for containerized deployment
- **Client Libraries**: Available for multiple languages

### Network Requirements
- **Ports**: 
  - 8000: Default HTTP/WebSocket port
  - 8001: Default RPC port (optional)
- **Protocol**: HTTP/HTTPS, WebSocket for real-time

## 2. Supported Operating Systems

SurrealDB officially supports:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04 LTS / 22.04 LTS / 24.04 LTS
- Arch Linux
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- Fedora 38+
- macOS 11+ (Big Sur and later)
- Windows 10/11

## 3. Installation

### Method 1: Install Script (Recommended)

#### Linux/macOS
```bash
# Install using official script
curl -sSf https://install.surrealdb.com | sh

# Add to PATH (add to ~/.bashrc or ~/.zshrc)
export PATH="$HOME/.surrealdb/bin:$PATH"

# Reload shell configuration
source ~/.bashrc  # or ~/.zshrc

# Verify installation
surreal version
```

#### Windows (PowerShell)
```powershell
# Install using PowerShell script
iwr https://install.surrealdb.com/ps1 -useb | iex

# Verify installation
surreal version
```

### Method 2: Package Managers

#### RHEL/CentOS/Rocky Linux/AlmaLinux
```bash
# Download latest release
SURREALDB_VERSION=$(curl -s https://api.github.com/repos/surrealdb/surrealdb/releases/latest | grep tag_name | cut -d '"' -f 4)
curl -LO "https://github.com/surrealdb/surrealdb/releases/download/${SURREALDB_VERSION}/surreal-${SURREALDB_VERSION}.linux-amd64.tgz"

# Extract and install
tar -xzf surreal-*.linux-amd64.tgz
sudo mv surreal /usr/local/bin/
sudo chmod +x /usr/local/bin/surreal

# Create systemd service
sudo tee /etc/systemd/system/surrealdb.service > /dev/null << 'EOF'
[Unit]
Description=SurrealDB Server
After=network.target

[Service]
Type=simple
User=surrealdb
Group=surrealdb
ExecStart=/usr/local/bin/surreal start --user root --pass root --bind 0.0.0.0:8000 file:/var/lib/surrealdb/data.db
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
EOF

# Create user and directories
sudo useradd -r -s /bin/false -d /var/lib/surrealdb surrealdb
sudo mkdir -p /var/lib/surrealdb
sudo chown -R surrealdb:surrealdb /var/lib/surrealdb
```

#### Debian/Ubuntu
```bash
# Method 1: Official installer
curl -sSf https://install.surrealdb.com | sh

# Method 2: Download binary
wget https://github.com/surrealdb/surrealdb/releases/latest/download/surreal-latest.linux-amd64.tgz
tar -xzf surreal-latest.linux-amd64.tgz
sudo mv surreal /usr/local/bin/
sudo chmod +x /usr/local/bin/surreal

# Add SurrealDB repository (when available)
# curl -fsSL https://pkg.surrealdb.com/pubkey.gpg | sudo apt-key add -
# echo "deb https://pkg.surrealdb.com/deb stable main" | sudo tee /etc/apt/sources.list.d/surrealdb.list
# sudo apt update && sudo apt install -y surrealdb
```

#### Arch Linux
```bash
# Install from AUR
yay -S surrealdb-bin

# Or using paru
paru -S surrealdb-bin

# Verify installation
surreal version
```

#### Alpine Linux
```bash
# Install dependencies
apk add --no-cache curl tar

# Download and install
SURREALDB_VERSION=$(curl -s https://api.github.com/repos/surrealdb/surrealdb/releases/latest | grep tag_name | cut -d '"' -f 4)
curl -LO "https://github.com/surrealdb/surrealdb/releases/download/${SURREALDB_VERSION}/surreal-${SURREALDB_VERSION}.linux-amd64.tgz"
tar -xzf surreal-*.linux-amd64.tgz
mv surreal /usr/local/bin/
chmod +x /usr/local/bin/surreal

# Create OpenRC service
tee /etc/init.d/surrealdb > /dev/null << 'EOF'
#!/sbin/openrc-run

description="SurrealDB Server"
command="/usr/local/bin/surreal"
command_args="start --user root --pass root --bind 0.0.0.0:8000 file:/var/lib/surrealdb/data.db"
command_user="surrealdb"
command_group="surrealdb"
pidfile="/run/surrealdb.pid"
command_background="yes"

depend() {
    need net
    after firewall
}

start_pre() {
    checkpath --directory --owner surrealdb:surrealdb --mode 0755 /var/lib/surrealdb
}
EOF

chmod +x /etc/init.d/surrealdb
```

#### macOS
```bash
# Method 1: Homebrew
brew install surrealdb/tap/surreal

# Method 2: Official installer
curl -sSf https://install.surrealdb.com | sh

# Method 3: MacPorts
sudo port install surrealdb
```

#### Windows
```powershell
# Method 1: Chocolatey
choco install surrealdb

# Method 2: Scoop
scoop bucket add surrealdb https://github.com/surrealdb/scoop-bucket.git
scoop install surrealdb

# Method 3: Direct download
Invoke-WebRequest -Uri "https://github.com/surrealdb/surrealdb/releases/latest/download/surreal-latest.windows-amd64.exe" -OutFile "surreal.exe"
Move-Item surreal.exe C:\Windows\System32\
```

### Method 3: Docker

```bash
# Run SurrealDB in Docker
docker run --rm -p 8000:8000 surrealdb/surrealdb:latest start

# With persistence
docker run -d \
  --name surrealdb \
  -p 8000:8000 \
  -v surrealdb_data:/data \
  surrealdb/surrealdb:latest \
  start --user root --pass root file:/data/database.db

# Docker Compose
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  surrealdb:
    image: surrealdb/surrealdb:latest
    container_name: surrealdb
    ports:
      - "8000:8000"
    volumes:
      - surrealdb_data:/data
    command: start --user root --pass root file:/data/database.db
    restart: unless-stopped

volumes:
  surrealdb_data:
EOF

docker-compose up -d
```

## 4. Configuration

### Command Line Options

```bash
# Start with file storage
surreal start --user root --pass root file:/path/to/data.db

# Start with memory storage
surreal start --user root --pass root memory

# Start with TiKV backend
surreal start --user root --pass root tikv://tikv-pd:2379

# Start with authentication disabled (development only)
surreal start --auth-level none file:/tmp/test.db

# Custom bind address and port
surreal start --bind 0.0.0.0:8080 --user admin --pass secretpass file:data.db

# Enable strict mode
surreal start --strict --user root --pass root file:data.db
```

### Configuration File

Create `/etc/surrealdb/config.toml`:
```toml
# SurrealDB configuration file

[server]
bind = "0.0.0.0:8000"
path = "file:/var/lib/surrealdb/data.db"

[auth]
user = "root"
pass = "your_secure_password"
level = "full"

[log]
level = "info"
format = "json"

[datastore]
strict = true
query_timeout = "5m"
transaction_timeout = "5m"

[capabilities]
allow_all = false
allow_scripting = true
allow_net = ["http://localhost", "https://api.example.com"]
```

### Environment Variables

```bash
# Set via environment
export SURREAL_USER="root"
export SURREAL_PASS="your_secure_password"
export SURREAL_PATH="file:/var/lib/surrealdb/data.db"
export SURREAL_BIND="0.0.0.0:8000"
export SURREAL_LOG="info"

# Start with environment configuration
surreal start
```

### Database Structure

```sql
-- Connect to SurrealDB
surreal sql --conn http://localhost:8000 --user root --pass root

-- Create namespace and database
USE NS production DB app;

-- Define schema
DEFINE TABLE user SCHEMAFULL;
DEFINE FIELD name ON user TYPE string;
DEFINE FIELD email ON user TYPE string ASSERT string::is::email($value);
DEFINE FIELD created ON user TYPE datetime DEFAULT time::now();
DEFINE INDEX idx_email ON user COLUMNS email UNIQUE;

-- Define permissions
DEFINE SCOPE account SESSION 24h
  SIGNIN (
    SELECT * FROM user WHERE email = $email AND crypto::argon2::compare(pass, $pass)
  )
  SIGNUP (
    CREATE user SET email = $email, pass = crypto::argon2::generate($pass)
  );
```

## 5. Service Management

### systemd Management

```bash
# Enable and start service
sudo systemctl daemon-reload
sudo systemctl enable --now surrealdb
sudo systemctl status surrealdb

# Start/stop/restart
sudo systemctl start surrealdb
sudo systemctl stop surrealdb
sudo systemctl restart surrealdb

# View logs
sudo journalctl -u surrealdb -f

# Check service status
sudo systemctl is-active surrealdb
```

### Manual Server Management

```bash
# Start server in foreground
surreal start --user root --pass root file:data.db

# Start server in background
nohup surreal start --user root --pass root file:data.db > surrealdb.log 2>&1 &

# Start with specific log level
surreal start --log debug --user root --pass root file:data.db

# Export and import data
surreal export --conn http://localhost:8000 --user root --pass root --ns test --db test export.sql
surreal import --conn http://localhost:8000 --user root --pass root --ns test --db test export.sql
```

### Client Connection

```bash
# Interactive SQL shell
surreal sql --conn http://localhost:8000 --user root --pass root --ns test --db test

# Execute query
echo "SELECT * FROM user;" | surreal sql --conn http://localhost:8000 --user root --pass root

# Pretty print output
surreal sql --conn http://localhost:8000 --user root --pass root --pretty

# Use WebSocket connection
surreal sql --conn ws://localhost:8000 --user root --pass root
```

## 6. Troubleshooting

### Common Issues

1. **Connection refused**:
```bash
# Check if service is running
sudo systemctl status surrealdb
ps aux | grep surreal

# Check port binding
sudo netstat -tlnp | grep 8000
sudo lsof -i :8000

# Check firewall
sudo firewall-cmd --list-ports
sudo ufw status
```

2. **Authentication errors**:
```bash
# Test with correct credentials
surreal sql --conn http://localhost:8000 --user root --pass root

# Reset root password (stop server first)
surreal start --user newuser --pass newpass file:data.db

# Check authentication level
surreal start --auth-level full file:data.db
```

3. **Database locked**:
```bash
# Check for running processes
ps aux | grep surreal

# Remove lock file if stale
rm -f /var/lib/surrealdb/data.db.lock

# Check file permissions
ls -la /var/lib/surrealdb/
```

4. **Performance issues**:
```bash
# Enable query analysis
surreal sql --conn http://localhost:8000
> INFO FOR DB;
> SHOW TABLES;

# Check index usage
> INFO FOR TABLE user;

# Monitor query execution
> EXPLAIN SELECT * FROM user WHERE email = 'test@example.com';
```

### Debug Mode

```bash
# Start with debug logging
surreal start --log trace --user root --pass root file:data.db

# Enable query logging
export SURREAL_LOG_QUERIES=true
surreal start

# Profile queries
surreal sql --conn http://localhost:8000
> PROFILE SELECT * FROM user;
```

## 7. Security Considerations

### Authentication and Authorization

```sql
-- Create namespace admin
DEFINE LOGIN admin ON NAMESPACE PASSWORD 'secure_password';

-- Create database admin
DEFINE LOGIN dbadmin ON DATABASE PASSWORD 'secure_password';

-- Create scoped access
DEFINE SCOPE user SESSION 24h
  SIGNIN (
    SELECT * FROM user WHERE email = $email AND crypto::argon2::compare(pass, $pass)
  )
  SIGNUP (
    CREATE user SET 
      email = $email, 
      pass = crypto::argon2::generate($pass),
      role = 'user'
  );

-- Define permissions
DEFINE TABLE post SCHEMAFULL
  PERMISSIONS
    FOR select WHERE published = true OR author = $auth.id
    FOR create WHERE $auth.role = 'user'
    FOR update WHERE author = $auth.id
    FOR delete WHERE author = $auth.id OR $auth.role = 'admin';
```

### Network Security

```bash
# Bind to localhost only
surreal start --bind 127.0.0.1:8000 --user root --pass root file:data.db

# Configure firewall
sudo firewall-cmd --permanent --add-port=8000/tcp
sudo firewall-cmd --reload

# Use reverse proxy with SSL
```

### nginx SSL Configuration

```nginx
server {
    listen 443 ssl http2;
    server_name surrealdb.example.com;
    
    ssl_certificate /etc/ssl/certs/surrealdb.crt;
    ssl_certificate_key /etc/ssl/private/surrealdb.key;
    
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Data Encryption

```bash
# Enable encryption at rest (coming soon)
surreal start --encryption-key "base64_encoded_key" file:data.db

# Use encrypted connections
surreal sql --conn https://surrealdb.example.com --user root --pass root
```

## 8. Performance Tuning

### Query Optimization

```sql
-- Create indexes for frequent queries
DEFINE INDEX idx_created ON user COLUMNS created;
DEFINE INDEX idx_email ON user COLUMNS email UNIQUE;
DEFINE INDEX idx_composite ON post COLUMNS author, created;

-- Use query hints
SELECT * FROM user WITH INDEX idx_email WHERE email = 'user@example.com';

-- Analyze query performance
EXPLAIN SELECT * FROM post WHERE author = user:123 ORDER BY created DESC LIMIT 10;
```

### Connection Pooling

```javascript
// JavaScript client example
import { Surreal } from 'surrealdb.js';

const db = new Surreal('http://localhost:8000', {
  pool: {
    min: 2,
    max: 10,
    idleTimeoutMillis: 30000
  }
});
```

### Resource Limits

```bash
# Set memory limits
surreal start --max-memory 4GB file:data.db

# Set query timeouts
surreal start --query-timeout 30s --transaction-timeout 5m file:data.db

# Limit concurrent connections
surreal start --max-connections 1000 file:data.db
```

### Storage Optimization

```bash
# Compact database
surreal compact --path file:data.db

# Use TiKV for horizontal scaling
surreal start tikv://pd1:2379,pd2:2379,pd3:2379

# Configure cache size
surreal start --cache-size 1GB file:data.db
```

## 9. Backup and Restore

### Database Backup

```bash
#!/bin/bash
# backup-surrealdb.sh

BACKUP_DIR="/var/backups/surrealdb"
DATE=$(date +%Y%m%d_%H%M%S)
DB_URL="http://localhost:8000"
DB_USER="root"
DB_PASS="root"

mkdir -p $BACKUP_DIR

# Export all namespaces and databases
surreal export \
  --conn $DB_URL \
  --user $DB_USER \
  --pass $DB_PASS \
  $BACKUP_DIR/surrealdb_backup_$DATE.sql

# Compress backup
gzip $BACKUP_DIR/surrealdb_backup_$DATE.sql

# Remove old backups
find $BACKUP_DIR -name "surrealdb_backup_*.sql.gz" -mtime +7 -delete

echo "Backup completed: $BACKUP_DIR/surrealdb_backup_$DATE.sql.gz"
```

### Automated Backup

```bash
# Add to crontab
sudo crontab -e

# Daily backup at 2 AM
0 2 * * * /opt/surrealdb/scripts/backup-surrealdb.sh

# Backup specific namespace/database
surreal export --conn http://localhost:8000 \
  --user root --pass root \
  --ns production --db app \
  backup.sql
```

### Restore Procedures

```bash
# Stop SurrealDB
sudo systemctl stop surrealdb

# Import backup
surreal import --conn http://localhost:8000 \
  --user root --pass root \
  backup.sql

# Restore specific namespace/database
surreal import --conn http://localhost:8000 \
  --user root --pass root \
  --ns production --db app \
  backup.sql

# Verify restore
surreal sql --conn http://localhost:8000 --user root --pass root
> USE NS production DB app;
> INFO FOR DB;
```

### Migration Tools

```bash
# Migrate from another database
surreal migrate postgres \
  --source "postgres://user:pass@localhost/db" \
  --target "http://localhost:8000" \
  --ns migrated --db postgres
```

## 10. System Requirements

### Minimum Requirements
- **CPU**: 1 core, 1.0 GHz
- **RAM**: 512MB
- **Storage**: 1GB
- **Network**: 100 Mbps

### Recommended Requirements
- **CPU**: 4+ cores, 2.0+ GHz
- **RAM**: 4GB+
- **Storage**: 100GB+ SSD
- **Network**: Gigabit Ethernet

### Production Requirements
- **CPU**: 8+ cores, 3.0+ GHz
- **RAM**: 16GB+
- **Storage**: 500GB+ NVMe
- **Network**: 10 Gbps for clusters

### Scaling Guidelines
- **Vertical**: Up to 128 cores, 1TB RAM
- **Horizontal**: TiKV backend for distributed storage
- **Connections**: 10,000+ concurrent WebSocket connections

## 11. Support

### Official Resources
- **Website**: https://surrealdb.com
- **GitHub**: https://github.com/surrealdb/surrealdb
- **Documentation**: https://surrealdb.com/docs
- **Discord**: https://discord.gg/surrealdb

### Community Support
- **Discord Server**: Very active community
- **GitHub Discussions**: https://github.com/surrealdb/surrealdb/discussions
- **Forum**: https://community.surrealdb.com
- **Twitter/X**: @surrealdb

## 12. Contributing

### How to Contribute
1. Fork the repository on GitHub
2. Create a feature branch
3. Submit pull request
4. Follow Rust coding standards
5. Include tests and documentation

### Development Setup
```bash
# Clone repository
git clone https://github.com/surrealdb/surrealdb.git
cd surrealdb

# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Build from source
cargo build --release

# Run tests
cargo test

# Run specific test
cargo test --package surrealdb --lib tests::parse
```

## 13. License

SurrealDB is licensed under the Business Source License 1.1 (BSL).

Key points:
- Free for most commercial use cases
- Source code available
- Converts to Apache 2.0 after 4 years
- Some restrictions on database-as-a-service offerings

## 14. Acknowledgments

### Credits
- **Tobie Morgan Hitchcock**: Co-founder and CEO
- **Jaime Morgan Hitchcock**: Co-founder and COO
- **SurrealDB Team**: Core development team
- **Community Contributors**: Feature development and testing

## 15. Version History

### Recent Releases
- **v1.x**: Production-ready release
- **v2.x**: Enhanced multi-model features (planned)

### Major Features by Version
- **v1.0**: First stable release with full SQL support
- **v0.8**: Added full-text search
- **v0.7**: Introduced graph queries

## 16. Appendices

### A. Quick Start Example

```javascript
// JavaScript client example
import { Surreal } from 'surrealdb.js';

const db = new Surreal('http://localhost:8000');

// Signin
await db.signin({
  user: 'root',
  pass: 'root',
});

// Select namespace and database
await db.use('test', 'test');

// Create record
await db.create('user', {
  name: 'John Doe',
  email: 'john@example.com',
  age: 30
});

// Query data
const users = await db.select('user');

// Advanced query
const results = await db.query(`
  SELECT * FROM user 
  WHERE age >= 18 
  ORDER BY created DESC 
  LIMIT 10
`);

// Real-time subscription
await db.live('user', (data) => {
  console.log('User changed:', data);
});
```

### B. SQL Examples

```sql
-- Connect via CLI
surreal sql --conn http://localhost:8000 --user root --pass root

-- Create records
CREATE user:john SET 
  name = "John Doe",
  email = "john@example.com",
  tags = ["developer", "admin"];

-- Relate records
RELATE user:john->likes->post:123 SET rating = 5, created = time::now();

-- Graph traversal
SELECT ->likes->post FROM user:john;

-- Aggregations
SELECT 
  count() as total,
  math::mean(age) as avg_age,
  array::distinct(tags) as all_tags
FROM user
GROUP BY country;

-- Full-text search
SELECT * FROM post WHERE title @@ "SurrealDB tutorial";

-- Geospatial queries
SELECT * FROM store WHERE location INSIDE {
  type: "Polygon",
  coordinates: [[[-0.1, 51.5], [-0.1, 51.6], [0.1, 51.6], [0.1, 51.5], [-0.1, 51.5]]]
};
```

### C. Schema Definition

```sql
-- Define table with schema
DEFINE TABLE product SCHEMAFULL;

-- Define fields
DEFINE FIELD name ON product TYPE string ASSERT $value != NONE;
DEFINE FIELD price ON product TYPE decimal ASSERT $value > 0;
DEFINE FIELD stock ON product TYPE int DEFAULT 0;
DEFINE FIELD categories ON product TYPE array;
DEFINE FIELD categories.* ON product TYPE record(category);

-- Define events
DEFINE EVENT product_low_stock ON product 
WHEN $after.stock < 10 AND $before.stock >= 10
THEN (
  CREATE notification SET 
    type = 'low_stock',
    product = $after.id,
    message = 'Product ' + $after.name + ' is low on stock'
);

-- Define functions
DEFINE FUNCTION fn::calculate_tax($price) {
  RETURN $price * 0.20;
};
```

### D. Client Libraries

```python
# Python client
import surrealdb

async def main():
    async with surrealdb.Surreal("ws://localhost:8000") as db:
        await db.signin({"user": "root", "pass": "root"})
        await db.use("test", "test")
        
        # Create user
        user = await db.create("user", {
            "name": "Jane Doe",
            "email": "jane@example.com"
        })
        
        # Query users
        users = await db.select("user")
        print(users)
```

```rust
// Rust client
use surrealdb::Surreal;
use surrealdb::sql::Thing;

#[tokio::main]
async fn main() -> surrealdb::Result<()> {
    let db = Surreal::new::<Ws>("127.0.0.1:8000").await?;
    
    db.signin(Root {
        username: "root",
        password: "root",
    }).await?;
    
    db.use_ns("test").use_db("test").await?;
    
    let user: Option<Record> = db
        .create("user")
        .content(User {
            name: "John",
            email: "john@example.com",
        })
        .await?;
    
    Ok(())
}
```

---

For more information and updates, visit https://github.com/howtomgr/surrealdb