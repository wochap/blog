+++
slug = "como-configurar-nvim-en-nixos"
title = "¿Cómo configurar nvim (mason.nvim y lazy.nvim) en NixOS con Home Manager?"
date = "2024-04-14"
description = "Aquí explicaré cómo configuro mi configuración personal de nvim que utiliza mason.nvim y lazy.nvim en NixOS."
summary = "Aquí explicaré cómo configuro mi configuración personal de nvim que utiliza mason.nvim y lazy.nvim en NixOS."
tags = [
  "nvim",
  "nixos",
]
showToc = true
showComments = true
+++

En esta guía te mostraré cómo hacer que [mason.nvim](https://github.com/williamboman/mason.nvim) y [lazy.nvim](https://github.com/folke/lazy.nvim) funcionen en NixOS. Sin embargo, si planeas hacer que tu configuración de nvim sea reproducible dentro de tu configuración de Nix, te sugiero que consultes lo siguiente: [nixvim](https://github.com/nix-community/nixvim).

## Muestrame el codigo!

Básicamente, solo necesitamos agregar algunas variables de entorno al ejecutable de nvim. En tu configuración de home-manager, agrega un nuevo módulo:

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

¡Exacto! Solo haz `rebuild` y cualquier distribución de nvim debería funcionar correctamente. Por cierto, aquí está mi configuración personal de NixOS para neovim:

* [modules/shared/programs/tui/neovim/default.nix](https://github.com/wochap/nix-config/blob/c43ed729d5f949368fdfa12207d17aef5888c1b2/modules/shared/programs/tui/neovim/default.nix)
* [modules/shared/programs/tui/neovim/options.nix](https://github.com/wochap/nix-config/blob/c43ed729d5f949368fdfa12207d17aef5888c1b2/modules/shared/programs/tui/neovim/options.nix)

## Reconocimientos

* [ayamir/nvimdots](https://github.com/ayamir/nvimdots)


