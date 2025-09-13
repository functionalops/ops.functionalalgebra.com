---
title: "Configuring Aspell with Custom Dictionaries in Nix"
summary: "Set up aspell with custom dictionaries for development environments using Nix user-level configuration and aspellWithDicts function."
tags: ["nix", "aspell", "development-environment", "spellcheck", "emacs"]
date: "2018-01-27"
mod: "2025-09-13"
---

## Updates Since 2016

Configuration Method Changes:
- `packageOverrides` deprecated in favor of overlays (still works but generates warnings)
- User config location changed from `~/.nixpkgs/config.nix` to `~/.config/nixpkgs/config.nix`
- Home Manager integration now preferred for user-level package management

Modern Nix Ecosystem:
- Flakes widely adopted for reproducible configurations
- `nix-env` discouraged in favor of declarative approaches
- New Nix commands available (`nix profile`, `nix shell`)

Technical Updates:
- `aspellWithDicts` function remains stable and unchanged
- Dictionary packaging and availability unchanged
- Environment variable handling (`ASPELL_CONF`) remains the same

Recommended Modern Approach:
- Use Home Manager for user-level package management
- Consider flakes for project-specific environments
- Use overlays instead of packageOverrides
- Replace `nix-env` with declarative configuration

This setup ensures reliable spell checking across your development tools while maintaining the reproducibility that makes Nix powerful for development environments.

---

The following is the original post content with some updates to the writing style to (hopefully) make it easier to read:

Need spell checking in your development environment? Here's how to configure `aspell` dictionaries through Nix for tools like flyspell in Emacs or Spacemacs.

## Configuration Approaches

You can configure aspell dictionaries in three ways:

**System Level Configuration**
Add aspell to your NixOS system configuration for server deployments or multi-user systems.

**User Level Configuration**
Install through your user profile for development machines and portable configurations.

**Environment Variable Method**
Set `ASPELL_CONF` with the `dict-dir` path in your shell configuration.

For development environments, user-level configuration works best. It keeps system configuration minimal while allowing you to copy your setup to remote development machines.

## User Level Nix Setup

Create or modify `${HOME}/.config/nixpkgs/config.nix` with your aspell configuration:

```nix
{ pkgs }:
let
  inherit (pkgs) aspellWithDicts buildEnv;

  # Configure aspell with desired dictionaries
  myaspell = aspellWithDicts (d: [d.en]);
  # Add more dictionaries: [d.en d.es d.fr]
in {
  allowUnfree = true;

  packageOverrides = pkgs: rec {
    myDevenv = buildEnv {
      name = "my-devenv";
      paths = with pkgs; [
        mtr
        tcpdump
        inetutils
        
        # Your custom aspell configuration
        myaspell
      ];
    };
  };
}
```

## Testing the Configuration

Install and verify your custom aspell setup:

```bash
$ nix-env -i myaspell

$ env | grep ASPELL
ASPELL_CONF=dict-dir /home/username/.nix-profile/lib/aspell

$ ls "$(readlink -f ~/.nix-profile/lib/aspell)/en_US*"
/nix/store/...-aspell-env/lib/aspell/en_US.multi
/nix/store/...-aspell-env/lib/aspell/en_US-variant_0.multi
/nix/store/...-aspell-env/lib/aspell/en_US-wo_accents-only.rws

$ aspell list <<<"some words one of which will be mispelledk"
mispelledk
```

## Modern Overlay Alternative

For newer nixpkgs versions, consider using overlays instead of `packageOverrides`:

```nix
{
  nixpkgs.overlays = [
    (self: super: {
      myaspell = super.aspellWithDicts (d: [d.en]);
    })
  ];
}
```

## Key Benefits

**Reproducible Development Environment**
Your aspell configuration follows you across machines and survives system rebuilds.

**Portable Configuration**
Copy your `config.nix` to any Nix-enabled system for consistent spell checking.

**Isolation from System Changes**
User-level configuration won't break when system packages change.

This setup ensures reliable spell checking across your development tools while maintaining the reproducibility that makes Nix powerful for development environments.

