---
title: nix-shell
publish: true
---

## what is nix shell


- nix shell会把我们带到一个新的shell里，这里只有必要的软件包可以调用去建造相应的derivation
   nix shell并不直接建造derivation,它只是做好建造所需的准备工作
   nix shell 使我们可以手动建造过程step by step
   在一nix表达上使用nix shell会返回相应的derivation然后进入一个bash shell,baseInputs src等环境变量都会被设置好
    
    ```sh
    $ nix-shell hello.nix
    [nix-shell]$ make
    bash: make: command not found
    [nix-shell]$ echo $baseInputs
    /nix/store/jff4a6zqi0yrladx3kwy4v6844s3swpc-gnutar-1.27.1 [...]
    ```
- 我们可以在此文件夹中source我们的builder.sh这样会为我们设置好PATH然后建造相应的包裹

## a builder for nix-shell

-  当前我们能source builder.sh是因为我们碰巧处于这个目录下，我们希望我们的builder.sh像nix-build过程一样被存储到nix store,我们可以在drv中设置新的attr:然后它将会被翻译成nix store中的位置并传递给nix-shell
- 对nix-shell来说我们更需要的是自动设置环境的部分，建造过程由于是手动的不那么重要
- 我们需要移除set -e：因为其会使得发生错误后退出当前shell,烦人

### 我们更新我们的autotools.nix为如下内容

```sh
➜  workround bat autotools.nix
───────┬────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: autotools.nix
───────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ pkgs: attrs:
   2   │   with pkgs;
   3   │   let defaultAttrs = {
   4   │     builder = "${bash}/bin/bash";
   5   │     args = [ ./builder.sh ];
   6   │     setup = ./setup.sh;
   7   │     baseInputs = [ gnutar gzip gnumake gcc coreutils gawk gnused gnugrep binutils.bintools patchelf findutils ];
   8   │     buildInputs = [];
   9   │     system = builtins.currentSystem;
  10   │   };
  11   │   in
  12   │ derivation (defaultAttrs // attrs)
───────┴────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
➜  workround
```

- 我们增加了setup这个attr,然后如上所述其会作为变量被传入shell

### setup.sh的内容

```sh
unset PATH
for p in $baseInputs $buildInputs; do
  export PATH=$p/bin${PATH:+:}$PATH
done

function unpackPhase() {
  tar -xzf $src

  for d in *; do
    if [ -d "$d" ]; then
      cd "$d"
      break
    fi
  done
}

function configurePhase() {
  ./configure --prefix=$out
}

function buildPhase() {
  make
}

function installPhase() {
  make install
}

function fixupPhase() {
  find $out -type f -exec patchelf --shrink-rpath '{}' \; -exec strip '{}' \; 2>/dev/null
}

function genericBuild() {
  unpackPhase
  configurePhase
  buildPhase
  installPhase
  fixupPhase
}
```

- 它将建造过程的环境准备设置为自动执行，而其他的环节分为几个特点的阶段(phase),将它们写成函数，只有在被调用的时候才会被执行

### 新的builder.sh的内容

```sh
➜  workround bat builder.sh
───────┬──────────────────────────────────────────────────────
       │ File: builder.sh
───────┼──────────────────────────────────────────────────────
   1   │ set -e
   2   │ source $setup
   3   │ genericBuild
───────┴──────────────────────────────────────────────────────
➜  workround
```

- 它会先执行我们的setup文件以自动设置环境，然后手动处发setup中的genericBuild函数

### 执行建造

```sh
➜  workround nix-shell hello.nix

[nix-shell:~/Documents/workround]$ echo $setup
/nix/store/3mygv5b5fx2pv2nyfs2dk2kain755dg2-setup.sh

[nix-shell:~/Documents/workround]$ source $setup

[nix-shell:~/Documents/workround]$ echo $PATH
/nix/store/nwfyy893ql9ld0y249a5miwjb2kp15y8-findutils-4.9.0/bin:/nix/store/5mm8iw0fdifw8jg7dphz70ly7087pv0j-patchelf-0.15.0/bin:/nix/store/shw0b6wv2xdvyj71b1fj147i83awrqfz-binutils-2.40/bin:/nix/store/4wwylhblws9na4ghmvsia38kimxl43g4-gnugrep-3.11/bin:/nix/store/figk1iqjicv30sa9qnvbbzdb81bzsh1c-gnused-4.9/bin:/nix/store/rw2r8jqvbsmpq5kmgwvkv9pd833k9h3z-gawk-5.2.1/bin:/nix/store/l8g5py3i39sq8afzi9vfmpw5igbqs84r-coreutils-9.1/bin:/nix/store/c22pksfcw1nnkcxlf5xm7ljxhngg3n65-gcc-wrapper-12.2.0/bin:/nix/store/914m27x3h5mfbxxdv8n5bcg1hqbl808d-gnumake-4.4.1/bin:/nix/store/4w0m22sx8yw9s1kw35f72cfak2qww8kh-gzip-1.12/bin:/nix/store/g1ajd0l91n1ryyw779wlfhf73imbh4cf-gnutar-1.34/bin
```

- 发现setup文件已被如期转移到store
- 环境source后环境变量也被设置好,一切如预料的一样接下来可以执行setup中设置好的建造函数,或者手动执行



