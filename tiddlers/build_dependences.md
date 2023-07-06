---
title: build dependences
publish: true
---

## 首先让我们分析刚刚建造的hello包的依赖

### 这之前有必要让我们看下现在的文件内容

```sh
workround bat hello.nix
───────┬──────────────────────────────────────────────────────
       │ File: hello.nix
───────┼──────────────────────────────────────────────────────
   1   │ let
   2   │   pkgs = import <nixpkgs> {};
   3   │   mkDerivation = import ./autotools.nix pkgs;
   4   │ in mkDerivation {
   5   │   name = "hello";
   6   │   src = ./hello-2.10.tar.gz;
   7   │ }
───────┴──────────────────────────────────────────────────────

───────┬────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: autotools.nix
───────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ pkgs: attrs:
   2   │   with pkgs;
   3   │   let defaultAttrs = {
   4   │     builder = "${bash}/bin/bash";
   5   │     args = [ ./builder.sh ];
   6   │     baseInputs = [ gnutar gzip gnumake gcc coreutils gawk gnused gnugrep binutils.bintools ];
   7   │     buildInputs = [];
   8   │     system = builtins.currentSystem;
   9   │   };
  10   │   in
  11   │   derivation (defaultAttrs // attrs)
───────┴────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

➜  workround bat builder.sh
───────┬────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: builder.sh
───────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ set -e
   2   │ unset PATH
   3   │ for p in $buildInputs; do
   4   │   export PATH=$p/bin${PATH:+:}$PATH
   5   │ done
   6   │
   7   │ tar -xf $src
   8   │
   9   │ for d in *; do
  10   │   if [ -d "$d" ]; then
  11   │     cd "$d"
  12   │     break
  13   │   fi
  14   │ done
  15   │
  16   │ ./configure --prefix=$out
  17   │ make
  18   │ make install
  19   │
───────┴────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
➜  workround
```

### DONE 在build时发现报错如下我们需要排查

```sh
➜  workround nix-build hello.nix
this derivation will be built:
  /nix/store/a254zr7593kjn2db9fnhdz95q58xi1lx-hello.drv
building '/nix/store/a254zr7593kjn2db9fnhdz95q58xi1lx-hello.drv'...

/nix/store/4yz8fqwrbmj6ycwl86lzdbn6w0k8qgrd-builder.sh: line 9: tar: No such file or directory
error: builder for '/nix/store/a254zr7593kjn2db9fnhdz95q58xi1l
x-hello.drv' failed with exit code 127;
       last 2 log lines:
       >
       > /nix/store/4yz8fqwrbmj6ycwl86lzdbn6w0k8qgrd-builder.sh: line 9: tar: No such file or directory
       For full logs, run 'nix log /nix/store/a254zr7593kjn2db9fnhdz95q58xi1lx-hello.drv'.
```

- 报错显示无法找到tar这个文件，那么意思就是说我们的PATH变量没有正确的设置
- 我们发现autotools.nix中的依赖写入的是baseInputs而我们shell脚本加入环境变量的变量只引用了buildInputs
- 当我们把script内容改为如下时建造成功
    
    ```sh
      set -e
    unset PATH
    for p in $buildInputs; do
      export PATH=$p/bin${PATH:+:}$PATH
    done
    
    for n in $baseInputs; do
      export PATH=$n/bin${PATH:+:}$PATH
    done
    
    echo $PATH
    
    tar -xf $src
    
    for d in *; do
      if [ -d "$d" ]; then
        cd "$d"
        break
      fi
    done
    
    ./configure --prefix=$out
    make
    make install
    ```
    

#### DONE 如何debug nix建造过程中的变量值？

可以使用nix shell手动复现建造过程

- 🔗To -> [what is nix shell](http://localhost:1313/posts/nix_shell/#1057b909-4e7f-41ab-8c9c-ad35fa5095a6)

### 在依赖中我们并未发现autotools.nix为其中的依赖

```sh
➜  workround nix-store -q --references /nix/store/5086c26jdq7xcyan64cp8w8gxyhbzyyh-hello.drv


/nix/store/18h8jz9vj7yb86lqahjq96bzyxg62kff-binutils-2.40.drv
/nix/store/1fqpzw45flwz5i5v4injpqh2fg4h8ij7-gawk-5.2.1.drv
/nix/store/6s9l92aczmdsqrvs4jbjf4c1dj23ljm5-bash-5.2-p15.drv
/nix/store/40wj3dkfky4xcn3arv82y5fkdyi9h1nx-gnumake-4.4.1.drv
/nix/store/5qsq9zk6a0lkj9j084s2860cxj43mqyl-builder.sh
/nix/store/dcc9bzivivd2yymvmj78mph1sfiv4nb6-gnugrep-3.11.drv
/nix/store/dn0kwiaw1yzq2i13919r29r0y4xgmn12-gnutar-1.34.drv
/nix/store/kpj5adpxdw639x8m8hwqwh2v41qddxa5-gzip-1.12.drv
/nix/store/y235v15p96w6xpxkkd207q06vvpsilra-coreutils-9.1.drv
/nix/store/p8vh8i416h2mxfvqjp2dxc8138zikzk0-gcc-wrapper-12.2.0.drv
/nix/store/slmc3fgfwq4yhafrd9g355i9klwzvgzx-gnused-4.9.drv
/nix/store/svc70mmzrlgq42m9acs0prsmci7ksh6h-hello-2.10.tar.gz
```

#### 假如我们修改autotools.nix的内容那么drv与output的hash值会变吗？

- autotools.nix
    
    ```nix
    pkgs: attr:
    with pkgs;
    let defaultAttrs = {
      builder = "${bash}/bin/bash";
      args = [ ./builder.sh ];
      baseInputs = [ gnutar gzip gnumake gcc coreutils gawk gnused gnugrep binutils.bintools ];
      buildInputs = [];
      system = builtins.currentSystem;
    };
    in
    derivation (defaultAttrs // attr)
    ```
    
- 我们把attrs变为了attr并执行nix build,发现hash值并未改变
    
    ```sh
      ➜  workround nvim autotools.nix
    ➜  workround nix-build hello.nix
    /nix/store/58nwz94j7knkgdvhfiv6q678fny4kzg8-hello
    ```
    
- 猜想:nix计算依赖时是依据最终够成的derivation这个set中的内容而不是形成这个set的文件



