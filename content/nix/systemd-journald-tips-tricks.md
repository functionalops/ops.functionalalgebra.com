---
aliases:
  - /2016/06/14/nixos-systemd-journald/
title: "Tips & Tricks for systemd and journald on NixOS"
summary: "Essential commands, configuration patterns, and logging strategies for managing systemd services and journald on NixOS systems"
tags: ["nixos", "systemd", "journald", "linux", "system-administration"]
date: "2016-06-14"
mod: "2025-09-13"
---

## Updates Since 2016

**NixOS Configuration Changes:**
- The `services.journald.rateLimitBurst` and `services.journald.rateLimitInterval` options remain available but now include more granular control
- NixOS has migrated to systemd 253+ with enhanced security features and cgroup v2 support
- The `systemd.services.<name>` syntax remains unchanged, but new options like `systemd.services.<name>.confinement` provide better sandboxing

**Systemd Evolution:**
- Systemd now includes portable services, user services improvements, and better container integration
- New unit types like `systemd-homed` for user management and `systemd-resolved` for DNS resolution
- Enhanced logging with structured logging support and better filtering capabilities

**Journald Improvements:**
- Forward Secure Sealing for tamper-proof logs
- Better storage management with automatic cleanup policies
- Enhanced filtering and export capabilities
- JSON output support for easier parsing

---

## Command Reference Quick Start

### Service Management Comparison

| SysVinit | Upstart | Systemd |
|----------|---------|---------|
| `/etc/init.d/service start` | `start service` | `systemctl start service` |
| `/etc/init.d/service stop` | `stop service` | `systemctl stop service` |
| `/etc/init.d/service restart` | `restart service` | `systemctl restart service` |
| `/etc/init.d/service status` | `status service` | `systemctl status service` |

## Understanding Systemd Unit Types

Systemd organizes system components into different unit types. Here are the essential ones:

### Services
Services manage long-running application processes. In NixOS, create a new systemd service like this:

```nix
systemd.services.myservice = {
  description = "My service handles data processing";
  after = [ "multi-user.target" ];
  wantedBy = [ "multi-user.target" ];
  path = [ pkgs.bash ];
  environment = {
    MY_SERVICE_HOME = "/my/path/here";
    MY_SERVICE_MAX_CONNS = toString myVar;
  };
  serviceConfig = {
    User = "myuser";
    ExecStart = path;
    Restart = "always";
  };
};
```

### Other Critical Unit Types

**Paths:** Monitor file system changes and trigger service actions using inotify.

**Slices:** Map to Linux Control Groups for resource management. The root slice is `-.slice`.

**Sockets:** Enable socket-based activation for network or IPC sockets.

**Targets:** Provide synchronization points during boot or state changes. Most services target `multi-user.target`.

**Timers:** Define periodic or event-based activation schedules.

## Essential Systemd Commands

### Unit Information Queries

```bash
# List dependencies for a unit
systemctl list-dependencies UNITNAME

# List active sockets
systemctl list-sockets

# Show running jobs
systemctl list-jobs

# Display all units and their states
systemctl list-unit-files

# Show loaded or active units
systemctl list-units
```

### Service Management

```bash
# Manage services (requires sudo)
sudo systemctl stop SERVICE
sudo systemctl start SERVICE
sudo systemctl restart SERVICE

# Query service status (no sudo needed)
systemctl status SERVICE
systemctl is-active SERVICE
systemctl show SERVICE
```

### Remote Management

Execute systemctl commands on remote hosts:

```bash
systemctl -H hostname status SERVICE
```

This works with any systemctl command, not just service-specific operations.

## Configuring Effective Logging

### Service Log Setup

Configure services to log to console for journalctl access. For Elasticsearch, use this logging configuration:

```yaml
rootLogger: INFO, console
logger:
  action: INFO
  com.amazonaws: WARN
appender:
  console:
    type: console
    layout:
      type: consolePattern
      conversionPattern: "[%d{ISO8601}][%-5p][%-25c] %m%n"
```

Then query logs with:

```bash
journalctl -f -u elasticsearch
```

## Mastering journalctl Commands

### Essential Log Queries

```bash
# Follow all log messages for elasticsearch service
journalctl -f -u elasticsearch

# Show last 1000 error messages (command exits automatically)
journalctl -fen1000 -u elasticsearch

# Follow kernel messages only
journalctl -k -f

# Show service logs since last boot
journalctl -b -u BLA

# Display all error messages since boot with extra details
journalctl -xab
```

### User Access Configuration

Add users to the `systemd-journal` group for log access:

```bash
groups USERNAME
```

Verify the user appears in the `systemd-journal` group.

## NixOS Journald Configuration

### Rate Limiting Tuning

Check current rate limiting settings:

```bash
sudo nixos-option services.journald.rateLimitBurst
sudo nixos-option services.journald.rateLimitInterval
```

Default values (as of NixOS 16.03+):
- **rateLimitBurst:** 100 messages
- **rateLimitInterval:** 10 seconds

### Optimizing for High-Traffic Servers

For servers with heavy logging, adjust these values:

```nix
services.journald.rateLimitBurst = 1000;
services.journald.rateLimitInterval = "1s";
```

This configuration allows 1000 messages per second per service.

### Disabling Rate Limiting

To remove all rate limiting:

```nix
services.journald.rateLimitInterval = "0";
```

**Warning:** Disabling rate limiting can impact system performance during log floods. Monitor disk usage and system load after making this change.

## Best Practices Summary

**Service Configuration:**
- Use descriptive service names and descriptions
- Set appropriate dependencies with `after` and `wantedBy`
- Configure proper restart policies for critical services

**Log Management:**
- Configure services to output logs to console for journalctl integration
- Tune rate limiting based on your application's logging patterns
- Regular log rotation and cleanup policies prevent disk space issues

**Security Considerations:**
- Run services with minimal privileges using dedicated users
- Use systemd's security features like `PrivateTmp` and `NoNewPrivileges`
- Monitor service status and logs for unusual activity

For comprehensive information about systemd units and journalctl options, consult `man systemctl` and `man journalctl`.

