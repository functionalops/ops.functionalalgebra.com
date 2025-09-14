---
aliases:
  - /2018/04/18/inspecting-nix-lambda-named-arguments/
title: "Inspecting Nix Lambda Function Arguments"
summary: "Learn how to discover what arguments a Nix function expects using builtins.functionArgs and understand how callPackage works under the hood."
tags: ["nix", "nixpkgs"]
date: "2018-04-18"
mod: "2025-09-13"
---

Ever wondered how `callPackage` magically knows which dependencies to pass to a package function? The secret lies in Nix's ability to inspect function arguments at runtime. Let's explore this powerful feature that makes nixpkgs dependency injection work seamlessly.

## Understanding Function Argument Inspection

When you work with nixpkgs, you often encounter functions that expect specific named arguments. Here's how to discover what those arguments are using `nix repl`:

```nix
nix-repl> f = import "${pkgs.path}/pkgs/servers/varnish"

nix-repl> f
«lambda @ /nix/store/.../pkgs/servers/varnish/default.nix:1:1»

nix-repl> builtins.functionArgs f
{ fetchurl = false; groff = false; libedit = false; libxslt = false; 
  makeWrapper = false; ncurses = false; pcre = false; pkgconfig = false; 
  python = false; pythonPackages = false; readline = false; stdenv = false; }
```

The `builtins.functionArgs` function returns an attribute set where:
- **Keys** represent the expected argument names
- **Values** indicate whether the argument has a default value (`true` means optional, `false` means required)

## How callPackage Uses This Information

The `callPackage` function leverages `builtins.functionArgs` to automatically provide the right dependencies. It intersects the function's expected arguments with the available packages:

```nix
nix-repl> builtins.intersectAttrs (builtins.functionArgs f) pkgs
{ fetchurl = «lambda @ .../fetchurl/default.nix:38:1»; 
  groff = «derivation /nix/store/...-groff-1.22.3.drv»; 
  libedit = «derivation /nix/store/...-libedit-20160903-3.1.drv»; 
  libxslt = «derivation /nix/store/...-libxslt-1.1.29.drv»; 
  makeWrapper = «derivation /nix/store/...-hook.drv»; 
  ncurses = «derivation /nix/store/...-ncurses-6.0-20171125.drv»; 
  pcre = «derivation /nix/store/...-pcre-8.41.drv»; 
  pkgconfig = «derivation /nix/store/...-pkg-config-0.29.2.drv»; 
  python = «derivation /nix/store/...-python-2.7.14.drv»; 
  pythonPackages = { ... }; 
  readline = «derivation /nix/store/...-readline-6.3p08.drv»; 
  stdenv = «derivation /nix/store/...-stdenv.drv»; }
```

This intersection creates the exact argument set needed to call the function, pulling each dependency from the nixpkgs attribute set.

## Practical Applications

**Exploring Package Dependencies**
When you want to understand what a package needs, check its function arguments:

```nix
builtins.functionArgs (import "<nixpkgs/pkgs/development/tools/misc/gdb>")
```

**Debugging Override Issues**
If package overrides fail, verify you're providing the correct argument names:

```nix
# Check what arguments the package actually expects
builtins.functionArgs pkgs.myPackage.override
```

**Creating Custom callPackage Functions**
Build your own dependency injection by combining `functionArgs` with `intersectAttrs`:

```nix
myCallPackage = f: args: 
  f (builtins.intersectAttrs (builtins.functionArgs f) (pkgs // args));
```

## Key Takeaways

The `builtins.functionArgs` function provides the foundation for nixpkgs' dependency management system. It enables:

- **Automatic dependency resolution** through callPackage
- **Runtime function introspection** for debugging
- **Custom dependency injection patterns** for advanced use cases

Next time you need to understand what arguments a Nix function expects, reach for `builtins.functionArgs`. This simple builtin unlocks powerful metaprogramming capabilities that make nixpkgs both flexible and maintainable.

