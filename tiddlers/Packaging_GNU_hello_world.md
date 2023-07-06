---
title: Packaging GNU hello world
publish: true
---

## 准备shell script与nix 文件

```shell
export PATH="$gnutar/bin:$gcc/bin:$gnumake/bin:$coreutils/bin:$gawk/bin:$gzip/bin:$gnugrep/bin:$gnused/bin:$bintools/bin"
tar -xzf $src
cd hello-2.10
./configure --prefix=$out
make
make install
```

```nix
with (import <nixpkgs> {});
derivation {
  name = "hello";
  builder = "${bash}/bin/bash";
  args = [ ./hello_builder.sh ];
  inherit gnutar gzip gnumake gcc coreutils gawk gnused gnugrep;
  bintools = binutils.bintools;
  src = ./hello-2.10.tar.gz;
  system = builtins.currentSystem;
}
```

### DONE stdenv.mkDerivation与Derivation的区别？


### TODO 我们必须每次都手写一个builder.sh手动传递依赖吗？

## build这个hello word程序

```sh
➜  workround nix-build hello.nix
/nix/store/1ag3mmn92l2wfw731n8ij6amfmkhvp9z-hello
```



