---
title: "Master Nix Fundamentals Through Practical Examples"
summary: "Learn Nix expression language basics with hands-on examples covering data types, functions, and common operations. Perfect for developers ready to move beyond theory."
tags: ["nix"]
layout: post
date: 2015-10-22
---

## Major Changes in Nix (2015-2025)

### Core System Changes
* **Nix Flakes** - Introduced with Nix 2.4 in November 2021, providing uniform structure for projects and version-pinned dependencies 
* **New nix command** - Unified command interface replacing separate utilities like nix-build, nix-shell 
* **Improved error messages** - Nix 2.4 brought significantly better error reporting with location information 
* **nix-repl deprecation** - The standalone nix-repl is now `nix repl` in the unified command structure

### Package Ecosystem Growth
* **Massive package expansion** - Nixpkgs grew to over 80,000 packages as of 2024 
* **Package management tools** - New utilities like nix-update for automated package maintenance
* **Channels evolution** - Channel system enhanced with better versioning (nixos-23.05 style naming)

### Development Workflow Changes
* **Flakes adoption** - Despite experimental status, flakes gained widespread community adoption for enhanced capabilities 
* **Reproducibility improvements** - Lock files provide better version pinning and reproducible installations 
* **Development environments** - Enhanced nix-shell workflows and development tooling

### Tooling and Ecosystem
* **NixOS modules system** - Enhanced configuration management since 2009
* **Community resources** - Wiki maintenance issues led to documentation reorganization
* **Integration tools** - Better support for non-NixOS systems and package managers

### Breaking Changes
* **Command interface** - Many traditional nix-* commands moved under unified `nix` command
* **Configuration syntax** - Flakes introduced new configuration patterns
* **Channel management** - Updated approaches to channel subscriptions and updates

---

Original post with minor updates:

Getting comfortable with Nix can feel overwhelming after reading the manual. You understand the concepts, but how do you actually *use* them? This cookbook bridges that gap with practical examples you can run and modify.

## What You'll Learn

This guide assumes you've read the [Nix Manual](http://nixos.org/nix/manual/) but need hands-on practice with:

* **Nix** - The functional package manager
* **NixOS** - The Linux distribution built on Nix
* **NixOps** - Deployment tool for NixOS systems

## Setup Requirements

You'll need Nix 1.9 or later. Most examples work with 1.8, but newer versions provide better error messages and performance.

Install the interactive REPL for testing expressions:

```bash
$ nix-env -i nix-repl
```

Or use the provided development environment:

```bash
$ nix-shell
```

This loads all necessary tools automatically.

## Core Language Elements

Master these fundamental data types and operations through interactive examples. Each section builds on previous concepts, so work through them in order.

**Start your learning journey with `nix-repl` open.**

### Data Types You'll Master

* **[Integers](https://github.com/functionalops/nix-cookbook/tree/master/basics/integers.nix)** - Math operations and number handling
* **[TODO Booleans](https://github.com/functionalops/nix-cookbook/tree/master/basics/booleans.nix)** - Logic operations and conditional expressions  
* **[Strings](https://github.com/functionalops/nix-cookbook/tree/master/basics/strings.nix)** - Text manipulation and interpolation
* **[TODO Paths](https://github.com/functionalops/nix-cookbook/tree/master/basics/paths.nix)** - File system navigation and path operations

### Advanced Operations

* **[TODO Files](https://github.com/functionalops/nix-cookbook/tree/master/basics/files.nix)** - Reading and processing file content
* **[Lists](https://github.com/functionalops/nix-cookbook/tree/master/basics/lists.nix)** - Collection operations and transformations
* **[Lambdas](https://github.com/functionalops/nix-cookbook/tree/master/basics/lambdas.nix)** - Function creation and application
* **[Concatenation](https://github.com/functionalops/nix-cookbook/tree/master/basics/concatenation.nix)** - Combining data structures
* **[Importing](https://github.com/functionalops/nix-cookbook/tree/master/basics/importing.nix)** - Code organization and module system

## Your Next Steps

Each example includes exercises to reinforce learning. Work through them systematically - the concepts build on each other.

**Found something confusing?** Submit [issues]({{ site.github.repo }}/issues) for clarification.

**Have improvements?** Send [pull requests]({{ site.github.repo }}/pulls) to help other learners.

The examples provide a foundation for real-world Nix usage. Once you're comfortable with these basics, you'll be ready to tackle complex system configurations and package definitions.


