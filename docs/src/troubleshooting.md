# Troubleshooting Guide

Common issues and their solutions when working with RDB.

---

## Table of Contents

1. [Installation Issues](#installation-issues)
2. [Server Issues](#server-issues)
3. [Query Issues](#query-issues)
4. [Authentication Issues](#authentication-issues)
5. [Performance Issues](#performance-issues)
6. [Data Issues](#data-issues)
7. [Configuration Issues](#configuration-issues)
8. [Getting Help](#getting-help)

---

## Installation Issues

### "cargo: command not found"

**Problem:** Rust toolchain not installed.

**Solution:**

```bash
# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Reload shell
source $HOME/.cargo/env

# Verify
cargo --version
```

### "error: linker `cc` not found"

**Problem:** C compiler not installed (required for some dependencies).

**Solution:**

```bash
# Ubuntu/Debian
sudo apt-get install build-essential

# macOS
xcode-select --install

# Windows
# Install Visual Studio Build Tools
```

### Build fails with "out of memory"

**Problem:** Not enough RAM for compilation.

**Solution:**

```bash
# Reduce parallel jobs
cargo build --release -j 1

# Or increase swap space
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

---

## Server Issues

### "Address already in use"

**Problem:** Port 8080 is already occupied.

**Solution:**

```bash
# Option 1: Use different port
rdb start --port 9090

# Option 2: Find and kill process using port
lsof -i :8080
kill -9 <PID>

# Option 3: Change default in config
rdb config set port 9090
rdb start
```

### "Permission denied" when starting server

**Problem:** Port < 1024 requires root privileges.

**Solution:**

```bash
# Use port >= 1024
rdb config set port 8080

# Or run with sudo (not recommended)
sudo rdb start
```

### Server crasheson start

**Problem:** Corrupted database or configuration.

**Solution:**

```bash
# Check logs
tail -f ~/.rdb/log/engine.log

# Verify files
ls -lh ~/.rdb/databases/

# Re-initialize (WARNING: Deletes data)
rm -rf ~/.rdb
rdb init
```

### "Cannot connect to server"

**Problem:** Server not running or firewall blocking.

**Solution:**

```bash
# Check if server is running
curl http://localhost:8080/

# Check firewall
sudo ufw status
sudo ufw allow 8080/tcp

# Bind to correct interface
rdb config set server.host 0.0.0.0
```

---

## Query Issues

### "Table not found"

**Problem:** Table doesn't exist in the database.

**Solution:**

```bash
# List databases and tables
rdb status

# Create table first
curl -X POST http://localhost:8080/query \
  -d '{"CreateTable": {...}}'
```

### "Invalid JSON"

**Problem:** Malformed JSON in request.

**Solution:**

```bash
# Validate JSON first
echo '{"Select": {...}}' | json_pp

# Use proper escaping
curl -X POST http://localhost:8080/query \
  -H "Content-Type: application/json" \
  -d '{
    "Select": {
      "database": "main",
      "from": "users",
      "columns": ["*"]
    }
  }'
```

### "Query timeout"

**Problem:** Query exceeds maximum execution time.

**Solution:**

```toml
# Increase timeout in config.toml
[limits]
max_query_time = 60  # seconds
```

### "JOINs are not yet implemented"

**Problem:** Attempting to use JOIN clause.

**Solution:**

```bash
# Workaround: Fetch data separately and join in application
# JOINs planned for v0.2.0
```

---

## Authentication Issues

### "Missing Authorization header"

**Problem:** No token provided in request.

**Solution:**

```bash
# 1. Login first
TOKEN=$(curl -X POST http://localhost:8080/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"password"}' \
  | jq -r '.token')

# 2. Include token in requests
curl -X POST http://localhost:8080/query \
  -H "Authorization: Bearer $TOKEN" \
  -d '{...}'
```

### "Invalid token format"

**Problem:** Token not formatted correctly.

**Solution:**

```bash
# Correct format
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Wrong formats
Authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...  # Missing "Bearer"
Authorization: Bearer: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...  # Extra colon
```

### "Token expired"

**Problem:** JWT token has passed expiration time.

**Solution:**

```bash
# Re-authenticate to get new token
curl -X POST http://localhost:8080/login \
  -d '{"username":"alice","password":"password"}'

# Or increase token lifetime
rdb config set auth.token_expiration 86400  # 24 hours
```

### "Insufficient permissions"

**Problem:** User role doesn't allow the operation.

**Solution:**

```bash
# Check user permissions
rdb access list --user alice

# Grant required permission
rdb access grant alice main ReadWrite
```

### "User not found"

**Problem:** Username doesn't exist.

**Solution:**

```bash
# Create user
rdb user add alice

# List all users
rdb user list
```

---

## Performance Issues

### Slow SELECT queries

**Causes & Solutions:**

1. **No index on query column**

   ```bash
   # Currently only primary keys indexed
   # Use primary key in WHERE clause
   ```

2. **Low buffer pool size**

   ```bash
   # Increase buffer pool
   rdb config set buffer_pool_size 2000
   ```

3. **Query cache disabled**

   ```bash
   # Enable caching
   rdb config set enable_query_cache true
   ```

4. **Large result set**
   ```bash
   # Use LIMIT
   {"Select": {..., "limit": 100}}
   ```

### Slow INSERT operations

**Causes & Solutions:**

1. **Large batches**

   ```bash
   # Split into smaller batches
   # Max: 10,000 rows per insert
   ```

2. **Compression overhead**

   ```bash
   # Increase threshold for small records
   rdb config set compression_threshold 128
   ```

3. **Frequent compaction**
   ```bash
   # Reduce compaction frequency
   rdb config set compact_threshold 20
   ```

### High memory usage

**Solutions:**

```bash
# Reduce buffer pool
rdb config set buffer_pool_size 100

# Reduce query cache
rdb config set query_cache_size 100

# Disable query cache
rdb config set enable_query_cache false
```

### High CPU usage

**Solutions:**

```bash
# Reduce compression
rdb config set compression_threshold 256

# Disable auto-compaction
rdb config set auto_compact false

# Reduce worker threads
rdb config set server.workers 2
```

---

## Data Issues

### "Page allocation failed"

**Problem:** Disk full or quota exceeded.

**Solution:**

```bash
# Check disk space
df -h

# Clean up old data
rm -rf ~/.rdb/log/*.log.old

# Move to larger partition
mv ~/.rdb /large_partition/.rdb
ln -s /large_partition/.rdb ~/.rdb
```

### Data corruption detected

**Problem:** Database file corrupted.

**Solution:**

```bash
# Try to recover
cp ~/.rdb/databases/main.db ~/.rdb/databases/main.db.backup

# Check logs for error details
tail -n 100 ~/.rdb/log/engine.log

# If unrecoverable, restore from backup
cp backup/main.db ~/.rdb/databases/main.db
```

### Lost data after crash

**Problem:** Dirty pages not flushed before crash.

**Solution:**

```bash
# Prevention: Enable more frequent flushing
rdb config set performance.flush_interval 60

# Recovery: Restore from backup
cp backup/main.db ~/.rdb/databases/main.db
```

---

## Configuration Issues

### "Config file not found"

**Problem:** config.toml missing or in wrong location.

**Solution:**

```bash
# Regenerate default config
rdb init

# Or specify custom location
export RDB_CONFIG=/path/to/config.toml
```

### "Invalid TOML syntax"

**Problem:** Syntax error in config.toml.

**Solution:**

```bash
# Validate TOML
cat ~/.rdb/config/config.toml | toml-cli validate

# Reset to defaults
rdb config reset
```

### Changes not taking effect

**Problem:** Configuration not reloaded.

**Solution:**

```bash
# Reload configuration
rdb config reload

# Or restart server
pkill rdb
rdb start
```

---

## Common Error Messages

### "Database locked"

**Cause:** Another process using database.  
**Solution:** Close other connections or restart server.

### "Too many open files"

**Cause:** OS file descriptor limit.  
**Solution:**

```bash
# Increase limit
ulimit -n 4096

# Permanent fix (Linux)
echo "* soft nofile 4096" >> /etc/security/limits.conf
echo "* hard nofile 8192" >> /etc/security/limits.conf
```

### "Broken pipe"

**Cause:** Connection closed by client.  
**Solution:** Check network connection, increase timeouts.

### "Cannot deserialize"

**Cause:** Invalid JSON structure.  
**Solution:** Check JSON schema against documentation.

---

## Debugging Tips

### Enable Debug Logging

```toml
[logging]
level = "debug"
log_to_file = true
log_file = "./rdb-debug.log"
```

### Check Detailed Logs

```bash
# Real-time monitoring
tail -f ~/.rdb/log/engine.log

# Search for errors
grep ERROR ~/.rdb/log/engine.log

# Last 100 lines
tail -n 100 ~/.rdb/log/engine.log
```

### Test Configuration

```bash
# Validate configuration
rdb config show

# Test with minimal config
rdb start --listen 127.0.0.1 --port 8080
```

### Isolate Issues

```bash
# Test with fresh database
rm -rf ~/.rdb/databases/test.db
rdb db create test

# Test single query
curl -X POST http://localhost:8080/query -d '{...}' -v
```

---

## Performance Profiling

### Query Performance

```bash
# Time a query
time curl -X POST http://localhost:8080/query -d '{...}'

# Check query execution plan (future)
{"Explain": {"Select": {...}}}
```

### Server Performance

```bash
# Monitor resources
top -p $(pgrep rdb)

# Memory usage
ps aux | grep rdb

# I/O statistics
iostat -x 1
```

---

## Getting Help

### Before Reporting Issues

1. ✅ Check this troubleshooting guide
2. ✅ Review documentation
3. ✅ Check existing GitHub issues
4. ✅ Enable debug logging
5. ✅ Try with minimal configuration

### How to Report Issues

Include this information:

```
**RDB Version:** (rdb --version)
**OS:** (uname -a)
**Rust Version:** (rustc --version)

**Problem:**
Clear description of the issue

**Steps to Reproduce:**
1. Step 1
2. Step 2
3. ...

**Expected Behavior:**
What should happen

**Actual Behavior:**
What actually happens

**Logs:**
Relevant log output

**Configuration:**
config.toml (sanitized)
```

### Support Channels

- 📖 **Documentation:** [docs/](https://github.com/yourusername/rdb/docs)
- 🐛 **Bug Reports:** [GitHub Issues](https://github.com/yourusername/rdb/issues)
- 💬 **Questions:** [GitHub Discussions](https://github.com/yourusername/rdb/discussions)
- 📧 **Email:** contact@muhammadfiaz.com

---

## Quick Fixes

| Problem            | Quick Fix                              |
| ------------------ | -------------------------------------- |
| Server won't start | `pkill rdb && rdb start`               |
| Can't connect      | `curl http://localhost:8080/`          |
| Auth issues        | `rdb user list`                        |
| Slow queries       | `rdb config set buffer_pool_size 2000` |
| High memory        | `rdb config set buffer_pool_size 100`  |
| Config issues      | `rdb config reset`                     |
| Data corruption    | `cp backup/main.db ~/.rdb/databases/`  |

---

## Next Steps

- **[Configuration](configuration.md)** - Tuning options
- **[CLI Reference](cli.md)** - Command help
- **[Architecture](architecture.md)** - Understanding internals
