---
title: "NixOS Configuration Examples for Modern Systems"
summary: "Learn NixOS configuration through practical examples covering system setup, modules, and option management. Updated for current NixOS versions and best practices."
tags: ["nixos", "linux"]
layout: page
---

## Terminology Mapping: Chef to NixOS

For those migrating from Chef, here's how concepts translate:

* `run_list` (Ruby DSL) → Nix expression
* `cookbook` (Ruby metadata DSL) → Nix expression (attrset)
* `recipe` (Ruby DSL) → Nix expression (module)
* `attributes` (JSON/Ruby) → Nix expression (attrset)
* `data_bag` (JSON/Ruby) → Nix expression (attrset)
* `environments` (JSON/Ruby) → Nix expression (attrset)
* `libraries` (Ruby) → Nix expression (functions and data)
* `resources` (Ruby) → Nix expression (functions and data)
* `files` → File
* `templates` (ERB) → Files using vars and expressions

## Basic System Configuration

```nix
{ config, pkgs, lib, ... }:
{
  # System basics
  time.timeZone = "UTC";
  
  # Network configuration
  networking = {
    hostName = "nixos-server";
    nameservers = [ "1.1.1.1" "8.8.8.8" ];
    firewall.enable = true;
  };

  # Package management
  nixpkgs.config.allowUnfree = true;

  # Time synchronization (NTP replaced by systemd-timesyncd by default)
  services.timesyncd = {
    enable = true;
    servers = [ 
      "0.pool.ntp.org"
      "1.pool.ntp.org" 
      "2.pool.ntp.org"
      "3.pool.ntp.org"
    ];
  };

  # User management
  users.users.alice = {
    isNormalUser = true;
    extraGroups = [ "wheel" "networkmanager" ];
    description = "Alice Johnson";
    home = "/home/alice";
  };

  # Security configuration
  security.pki.certificateFiles = [ ./company_ca.crt ];
  
  # Kernel parameters
  boot.kernel.sysctl = {
    "net.ipv4.tcp_keepalive_time" = 1500;
    "net.core.default_qdisc" = "fq";
  };

  # System state version (important for upgrades)
  system.stateVersion = "24.05";
}
```

**Reference:** [NixOS Configuration Syntax](https://nixos.org/manual/nixos/stable/index.html#sec-configuration-syntax)

## Service Configuration with Modules

```nix
{
  # Database services
  services.mysql = {
    enable = true;
    package = pkgs.mariadb;
    dataDir = "/var/lib/mysql";
    settings = {
      mysqld = {
        port = 3306;
        bind-address = "127.0.0.1";
      };
    };
    initialScript = pkgs.writeText "mysql-init.sql" ''
      CREATE DATABASE IF NOT EXISTS myapp;
      CREATE USER IF NOT EXISTS 'appuser'@'localhost' IDENTIFIED BY 'secretpassword';
      GRANT ALL PRIVILEGES ON myapp.* TO 'appuser'@'localhost';
      FLUSH PRIVILEGES;
    '';
  };

  # Web server
  services.nginx = {
    enable = true;
    virtualHosts."example.com" = {
      root = "/var/www/example.com";
      locations."/" = {
        tryFiles = "$uri $uri/ =404";
      };
    };
  };
}
```

## Exploring Configuration Options

```bash
# Modern command (replaces nixos-option)
$ nix-instantiate --eval -E '(import <nixpkgs/nixos> {}).options.services.mysql.enable'

# Search options online
$ nix search nixpkgs mysql

# Get option documentation
$ man configuration.nix
```

**Online Reference:** [NixOS Options Search](https://search.nixos.org/options)

## Writing Custom Modules

```nix
{ config, lib, pkgs, ... }:

with lib;

let
  cfg = config.services.myapp;
in
{
  options.services.myapp = {
    enable = mkEnableOption "MyApp service";
    
    port = mkOption {
      type = types.port;
      default = 8080;
      description = "Port for MyApp to listen on";
    };
    
    settings = mkOption {
      type = types.attrs;
      default = {};
      example = { debug = true; };
      description = "Configuration settings for MyApp";
    };
    
    dataDir = mkOption {
      type = types.path;
      default = "/var/lib/myapp";
      description = "Directory to store MyApp data";
    };
  };

  config = mkIf cfg.enable {
    systemd.services.myapp = {
      description = "MyApp Service";
      wantedBy = [ "multi-user.target" ];
      after = [ "network.target" ];
      
      serviceConfig = {
        Type = "simple";
        User = "myapp";
        Group = "myapp";
        ExecStart = "${pkgs.myapp}/bin/myapp --port ${toString cfg.port}";
        Restart = "always";
        WorkingDirectory = cfg.dataDir;
      };
    };

    users.users.myapp = {
      isSystemUser = true;
      group = "myapp";
      home = cfg.dataDir;
      createHome = true;
    };
    
    users.groups.myapp = {};

    # Open firewall port if needed
    networking.firewall.allowedTCPPorts = mkIf cfg.enable [ cfg.port ];
  };
}
```

## Modern NixOS Workflow

### Using Flakes (Recommended)

```nix
# flake.nix
{
  description = "My NixOS configuration";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-24.05";
  };

  outputs = { self, nixpkgs }: {
    nixosConfigurations.myhost = nixpkgs.lib.nixosSystem {
      modules = [
        ./configuration.nix
      ];
    };
  };
}
```

### Build and Deploy

```bash
# Traditional approach
$ sudo nixos-rebuild switch

# With flakes
$ sudo nixos-rebuild switch --flake .#myhost

# Test configuration without switching
$ sudo nixos-rebuild test --flake .#myhost
```

## Key Changes Since 2015

* **Flakes support** for better dependency management
* **Unified `nix` command** replaces many individual tools
* **Better systemd integration** with improved service definitions
* **Enhanced security defaults** and hardening options
* **Module system improvements** with better type checking
* **Updated package ecosystem** with 80,000+ packages available

