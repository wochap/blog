+++
title = "How to setup nvim (mason.nvim and lazy.nvim) on NixOS with Home Manager?"
date = "2024-04-14"
description = "Here I'll explain how I setup my personal nvim config that uses mason.nvim and lazy.nvim on NixOS."
summary = "Here I'll explain how I setup my personal nvim config that uses mason.nvim and lazy.nvim on NixOS."
tags = [
  "nvim",
  "nixos",
]
toc = true
+++

In this guide I'll show you how to make [mason.nvim](https://github.com/williamboman/mason.nvim) and [lazy.nvim](https://github.com/folke/lazy.nvim) work on NixOS, however if you plan on making your nvim config reproducible within your nix config, I suggest you to check the following: [nixvim](https://github.com/nix-community/nixvim).

## Show me the code!

Basically we only need to prepend some env variables to the nvim executable, in your home-manager config add a new module:

```nix
{ config, lib, pkgs, ... }:

with lib;
let
  build-dependent-pkgs = with pkgs;
    [
      acl
      attr
      bzip2
      curl
      libsodium
      libssh
      libxml2
      openssl
      stdenv.cc.cc
      systemd
      util-linux
      xz
      zlib
      zstd
      glib
      libcxx
    ];

  makePkgConfigPath = x: makeSearchPathOutput "dev" "lib/pkgconfig" x;
  makeIncludePath = x: makeSearchPathOutput "dev" "include" x;

  nvim-depends-library = pkgs.buildEnv {
    name = "nvim-depends-library";
    paths = map lib.getLib build-dependent-pkgs;
    extraPrefix = "/lib/nvim-depends";
    pathsToLink = [ "/lib" ];
    ignoreCollisions = true;
  };
  nvim-depends-include = pkgs.buildEnv {
    name = "nvim-depends-include";
    paths = splitString ":" (makeIncludePath build-dependent-pkgs);
    extraPrefix = "/lib/nvim-depends/include";
    ignoreCollisions = true;
  };
  nvim-depends-pkgconfig = pkgs.buildEnv {
    name = "nvim-depends-pkgconfig";
    paths = splitString ":" (makePkgConfigPath build-dependent-pkgs);
    extraPrefix = "/lib/nvim-depends/pkgconfig";
    ignoreCollisions = true;
  };
  buildEnv = [
    "CPATH=${config.home.profileDirectory}/lib/nvim-depends/include"
    "CPLUS_INCLUDE_PATH=${config.home.profileDirectory}/lib/nvim-depends/include/c++/v1"
    "LD_LIBRARY_PATH=${config.home.profileDirectory}/lib/nvim-depends/lib"
    "LIBRARY_PATH=${config.home.profileDirectory}/lib/nvim-depends/lib"
    "NIX_LD_LIBRARY_PATH=${config.home.profileDirectory}/lib/nvim-depends/lib"
    "PKG_CONFIG_PATH=${config.home.profileDirectory}/lib/nvim-depends/pkgconfig"
  ];
in {
  home.packages = with pkgs; [
    patchelf
    nvim-depends-include
    nvim-depends-library
    nvim-depends-pkgconfig
    ripgrep
  ];
  home.extraOutputsToInstall = "nvim-depends";
  home.shellAliases.nvim = (concatStringsSep " " buildEnv)
    + " SQLITE_CLIB_PATH=${pkgs.sqlite.out}/lib/libsqlite3.so " + "nvim";

  programs.neovim = {
    enable = true;
    package = pkgs.neovim;

    withNodeJs = true;
    withPython3 = true;
    withRuby = true;

    extraPackages = with pkgs;
      [
        doq
        sqlite
        cargo
        clang
        cmake
        gcc
        gnumake
        ninja
        pkg-config
        yarn
      ];

    extraLuaPackages = ls: with ls; [ luarocks ];
  };
};
```

Yeah, that's it, just rebuild and any nvim distro should just work, BTW, here's my personal nixos config for neovim:

* [modules/shared/programs/tui/neovim/default.nix](https://github.com/wochap/nix-config/blob/c43ed729d5f949368fdfa12207d17aef5888c1b2/modules/shared/programs/tui/neovim/default.nix)
* [modules/shared/programs/tui/neovim/options.nix](https://github.com/wochap/nix-config/blob/c43ed729d5f949368fdfa12207d17aef5888c1b2/modules/shared/programs/tui/neovim/options.nix)

## Acknowledgement

* [ayamir/nvimdots](https://github.com/ayamir/nvimdots)


