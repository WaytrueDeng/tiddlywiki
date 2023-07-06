---
title: convenient derivation function
publish: true
---

## autotools.nix:将拥有共同特点的derivation部分抽象出来

- autotoos.nix:

```nix
pkgs: attrs:
  with pkgs;
  let defaultAttrs = {
    builder = "${bash}/bin/bash";
    args = [ ./builder.sh ];
    baseInputs = [ gnutar gzip gnumake gcc coreutils gawk gnused gnugrep binutils.bintools ];
    buildInputs = [];
    system = builtins.currentSystem;
  };
  in
  derivation (defaultAttrs // attrs)
```

## 更新一下hello.nix的模版

```nix
let
  pkgs = import <nixpkgs> {};
  mkDerivation = import ./autotools.nix pkgs;
in mkDerivation {
  name = "hello";
  src = ./hello-2.10.tar.gz;
}
```

### mkDerivation变量的作用是？

- 🔗By <- stdenv.mkDerivation与Derivation的区别?
- 首先通过import引入了在autotools.nix中的表达式，然后我们通过应用pkgs的参数返回一个接受attr的函数
- 这样我们就能复用大部分相同的信息

### 如何理解import autotools.nix的作用？

- 把autotools.nix理解为一个函数，它的函数名是文件名,它接受一个叫做pkgs的参数，然后返回另一个接受attr参数的函数并将attr作为参数传递给derivavtion函数
- 当我们import时相当于执行了autotools函数然后其后的pkgs就是相应的第一个参数



