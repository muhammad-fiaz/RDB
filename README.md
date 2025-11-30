<div align="center">
<h1>RDB - Relational Database with JSON Query Language</h1>

<a href="https://github.com/muhammad-fiaz/rdb"><img src="https://img.shields.io/github/stars/muhammad-fiaz/rdb" alt="GitHub stars"></a>
<a href="https://github.com/muhammad-fiaz/rdb/issues"><img src="https://img.shields.io/github/issues/muhammad-fiaz/rdb" alt="GitHub issues"></a>
<a href="https://github.com/muhammad-fiaz/rdb/pulls"><img src="https://img.shields.io/github/issues-pr/muhammad-fiaz/rdb" alt="GitHub pull requests"></a>
<a href="https://github.com/muhammad-fiaz/rdb"><img src="https://img.shields.io/github/last-commit/muhammad-fiaz/rdb" alt="GitHub last commit"></a>
<a href="https://github.com/muhammad-fiaz/rdb/releases"><img src="https://img.shields.io/github/v/release/muhammad-fiaz/rdb" alt="GitHub release"></a>
<a href="https://github.com/muhammad-fiaz/rdb"><img src="https://img.shields.io/github/license/muhammad-fiaz/rdb" alt="License"></a>
<a href="https://www.rust-lang.org/"><img src="https://img.shields.io/badge/rust-1.70%2B-orange" alt="Rust"></a>
<a href="https://github.com/muhammad-fiaz/RDB/actions/workflows/deploy-docs.yml"><img src="https://github.com/muhammad-fiaz/RDB/actions/workflows/deploy-docs.yml/badge.svg" alt="Build Docs"></a>

<p><em>High-performance, JSON-based relational database built entirely in Rust.</em></p>

**📚 [Documentation](https://muhammad-fiaz.github.io/RDB/) | [Quick Start](https://muhammad-fiaz.github.io/RDB/introduction.html)**

</div>

Welcome to **RDB** - a high-performance, JSON-based relational database built entirely in Rust!

RDB is a modern relational database that combines the power of traditional SQL databases with the simplicity of JSON APIs. Instead of writing SQL strings, you send structured JSON objects to describe your queries.

## 📥 Installation

Download the latest release from [GitHub Releases](https://github.com/muhammad-fiaz/rdb/releases).

### Quick Install

| Platform | Method | Command |
|----------|--------|---------|
| **Linux (x86_64)** | Direct Download | `wget https://github.com/muhammad-fiaz/rdb/releases/latest/download/rdb-linux-x86_64 && chmod +x rdb-linux-x86_64 && sudo mv rdb-linux-x86_64 /usr/local/bin/rdb` |
| **macOS (Intel)** | Direct Download | `curl -L https://github.com/muhammad-fiaz/rdb/releases/latest/download/rdb-macos-x86_64 -o rdb && chmod +x rdb && sudo mv rdb /usr/local/bin/` |
| **macOS (Apple Silicon)** | Direct Download | `curl -L https://github.com/muhammad-fiaz/rdb/releases/latest/download/rdb-macos-aarch64 -o rdb && chmod +x rdb && sudo mv rdb /usr/local/bin/` |
| **Windows (x64)** | Direct Download | Download `rdb-windows-x86_64.exe` and add to PATH |
| **Any Platform** | From Source | `cargo install --git https://github.com/muhammad-fiaz/rdb` |

### Detailed Installation

#### Linux
```bash
# Download
wget https://github.com/muhammad-fiaz/rdb/releases/latest/download/rdb-linux-x86_64

# Make executable
chmod +x rdb-linux-x86_64

# Move to PATH
sudo mv rdb-linux-x86_64 /usr/local/bin/rdb

# Verify
rdb --version
```

#### macOS
```bash
# Intel Macs
curl -L https://github.com/muhammad-fiaz/rdb/releases/latest/download/rdb-macos-x86_64 -o rdb

# Apple Silicon Macs
curl -L https://github.com/muhammad-fiaz/rdb/releases/latest/download/rdb-macos-aarch64 -o rdb

# Make executable and move to PATH
chmod +x rdb
sudo mv rdb /usr/local/bin/

# Verify
rdb --version
```

#### Windows
```powershell
# Download rdb-windows-x86_64.exe from releases

# Move to a directory in your PATH, e.g.:
Move-Item rdb-windows-x86_64.exe C:\Windows\System32\rdb.exe

# Or create a dedicated directory:
New-Item -ItemType Directory -Path "C:\Program Files\RDB"
Move-Item rdb-windows-x86_64.exe "C:\Program Files\RDB\rdb.exe"

# Add to PATH
$env:Path += ";C:\Program Files\RDB"

# Verify
rdb --version
```

## 🚀 Quick Start

```bash
# Initialize RDB
rdb init

# Start server
rdb start

# Create your first table
curl -X POST http://localhost:8080/query \
  -H "Content-Type: application/json" \
  -d '{
    "CreateTable": {
      "database": "main",
      "table": "users",
      "columns": [
        {"name": "id", "type": "int", "primary_key": true},
        {"name": "name", "type": "string"}
      ]
    }
  }'
```

## ✨ Features

- ✅ Complete CRUD operations (CREATE, SELECT, INSERT, UPDATE, DELETE, DROP)
- ✅ 8 WHERE operators (=, !=, >, <, >=, <=, LIKE, IN)
- ✅ ORDER BY, LIMIT, OFFSET support
- ✅ Batch operations
- ✅ Multi-layer caching (Query cache + Buffer pool + B+ tree)
- ✅ JWT authentication with role-based access control
- ✅ Dynamic configuration via config.toml
- ✅ CLI management tools
- ✅ Database auto-discovery
- ✅ Comprehensive documentation

## 📊 Performance

- ⚡ **100,000 queries/second** (cached SELECT)
- 📊 **90-98% cache hit rate**
- 🚀 **10-50x faster** JSON parsing vs SQL
- 💾 **O(log N)** indexed lookups

## 📊 System Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **RAM** | 512 MB | 2 GB+ |
| **Disk** | 100 MB | 1 GB+ |
| **CPU** | 1 core | 4+ cores |
| **OS** | Linux 3.2+, macOS 10.12+, Windows 10+ | Latest |

## 🔧 Troubleshooting

### Common Issues

#### "Address already in use"
```bash
# Use different port
rdb start --port 9090
```

#### "Permission denied"
```bash
# On Linux/macOS, ensure executable
chmod +x rdb

# On Windows, run as Administrator
```

#### "Cannot connect to server"
```bash
# Check if server is running
curl http://localhost:8080/

# Check firewall settings
```

### Get Help
- 📖 **Docs:** https://muhammad-fiaz.github.io/RDB/
- 🐛 **Issues:** https://github.com/muhammad-fiaz/rdb/issues
- 💬 **Discussions:** https://github.com/muhammad-fiaz/rdb/discussions

If you encounter any database errors, please report them at: https://github.com/muhammad-fiaz/rdb/issues

## 📝 Changelog

Visit [GitHub Releases](https://github.com/muhammad-fiaz/rdb/releases) for detailed changes.

## 📄 License

Licensed under the Apache License 2.0 - see [LICENSE](https://github.com/muhammad-fiaz/rdb/blob/main/LICENSE) for details.


