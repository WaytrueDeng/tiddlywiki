---
title: nixCreatRepo
publish: true
---

## Repositories

- Repository就是放软件的仓库

### 什么是single repository?

- 与debian系将软件分散到几个小仓库不同，nix像gentoo一样将所有软件放在一个仓库
- nix的仓库就是nixpkgs,nixpkgs用一个top-level的表达式import所有其他的软件的nix表达式，形成一个set:包名与表达式的键值对合集
- nix语言很lazy只会执行真正需要的部分，所以nixpkgs才会庞大却轻盈

## packging graphviz

- 我们可以像之前那样写一个表达示建造它,但是有几个问题
    
    ```nix
    let
      pkgs = import <nixpkgs> {};
      mkDerivation = import ./autotools.nix pkgs;
    in mkDerivation {
      name = "graphviz";
      src = ./graphviz-2.49.3.tar.gz;
    }
    ```
    

### 如何添加额外的库使之支持更多的格式？

- graphviz使用的是pkg-config工具传递参数给compiler编译器

#### 在nix中如何让编译器找到库的位置

- pkg-cofnig通过查询pc文件以知道第三方库的位置 =>
    
    pc文件的位置是通过PKG_CONFIG_PATH变量所储存的 =>
    
    nix没有统一的PKG_CONFIG_PATH,所以我们需要添加每个依赖的pc文件位置到此变量中
    
- 标准的linux pc文件储存在
    
    ```sh
      /usr/share/pkgconfig
    ├── adwaita-icon-theme.pc
    ├── bigreqsproto.pc
    ├── compositeproto.pc
    ├── damageproto.pc
    ├── dmxproto.pc
    ...
    ```
    
- 探寻一下pc文件的内容
    
    ```pc
      prefix=/usr
    exec_prefix=${prefix}
    libdir=${prefix}/lib
    includedir=${prefix}/include
    targets=broadway wayland x11
    
    gtk_binary_version=3.0.0
    gtk_host=x86_64-linux
    
    Name: GTK+
    Description: GTK+ Graphical UI Library
    Version: 3.24.37
    Requires: gdk-3.0 atk >= 2.35.1  cairo >= 1.14.0 cairo-gobject >= 1.14.0 gdk-pixbuf-2.0 >= 2.30.0 gio-2.0 >= 2.57.2
    Requires.private: atk atk-bridge-2.0 wayland-client >= 1.14.91 xkbcommon >= 0.2.0 wayland-cursor >= 1.14.91 wayland-egl epoxy >= 1.4 fribidi >= 0.19.7 pangoft2 gio-unix-2.0 >= 2.57.2
    Libs: -L${libdir} -lgtk-3
    Cflags: -I${includedir}/gtk-3.0
    ```
    

##### nix中pc文件的存放位置在哪？

```sh
➜  store exa yhpwdgqsrfm5r4x7ycbgj04aq7063rli-python3-3.10.11/lib
libpython3.10.so  libpython3.10.so.1.0  libpython3.so  pkgconfig  python3.10
```

分散在每个out路径的lib里,而不是有一个统一的/usr/lib/pkgconfig

#### 将所依赖的库加入到环境变量中,更新setup.sh

```sh
for p in $baseInputs $buildInputs; do
  if [ -d $p/bin ]; then
    export PATH="$p/bin${PATH:+:}$PATH"
  fi
  if [ -d $p/lib/pkgconfig ]; then
    export PKG_CONFIG_PATH="$p/lib/pkgconfig${PKG_CONFIG_PATH:+:}$PKG_CONFIG_PATH"
  fi
done
```

- 这个脚本遍例所有依赖并将它们的bin目录加到PATH中，将它们的pkgconfig目录加到PKG_CONFIG_PATH中

#### 改进我们的nix表达式加入所选用的库

```nix
let
  pkgs = import <nixpkgs> {};
  mkDerivation = import ./autotools.nix pkgs;
in mkDerivation {
  name = "graphviz";
  src = ./graphviz-2.49.3.tar.gz;
  buildInputs = with pkgs; [
    pkg-config
    (pkgs.lib.getLib gd)
    (pkgs.lib.getDev gd)
  ];
}
```

##### 小括号在nix表达式中的意义是什么？

如 `(pkgs.lib.getDev gd)` 所示其中的小括号含义是什么?

```nix
nix-repl> mul = a: b: a*b
nix-repl> mul
«lambda»
nix-repl> mul 3
«lambda»
nix-repl> mul 3 4
12
nix-repl> mul (6+7) (8+9)
221
```

- 括号在nix中用来call funcion的
    
    ```sh
      nix-repl> mul 3 4
    12
    
    nix-repl> (mul 3) 4
    12
    
    nix-repl> (mul 3 4)
    12
    nix-repl> mul 3 (4)
    12
    
    nix-repl> mul (3) (4)
    12
    
    nix-repl> (mul) (3) (4)
    12
    
    nix-repl> mul (3 4)
    «lambda @ «string»:1:5» #这样是不行的
    ```
    

##### 为什么gd要分别得到两个？

因为gd这个包用的是split outputs,有多个out路径，我们看下gd这个drv

```sh
{
  "/nix/store/ba6fybvnj12z78rbjj3yp87pmf2d1rdg-gd-2.3.3.drv": {
    "args": [
      "-e",
      "/nix/store/6xg259477c90a229xwmb53pdfkn6ig3g-default-builder.sh"
    ],
    "builder": "/nix/store/h6rxqqi4m0vipgvv9czpmwvifdjawb14-bash-5.2-p15/bin/bash",
    "env": {
      "NIX_HARDENING_ENABLE": "fortify stackprotector pic strictoverflow relro bindnow",
      "__structuredAttrs": "",
      "bin": "/nix/store/gxwps8pxk904k76syj15fjal7gppymar-gd-2.3.3-bin",
      "buildInputs": "/nix/store/5604wq5idbgniqchzal2pjmjgv9ns1kb-zlib-1.2.13-dev /nix/store/r9amx44n3643i1snpw6rxak3ifr4ca6s-fontconfig-2.14.0-dev /nix/store/bihr8pvzc6ai2ajss6kpd7knmn8jsjnx-freetype-2.13.0-dev /nix/store/slr08dnahrwpl0vnjqw0vz6cfgshy51p-libpng-apng-1.6.39-dev /nix/store/wvb3ggdvgdxhh93d382wv00ycmnd6ind-libjpeg-turbo-2.1.5.1-dev /nix/store/qaghh6kiw1w9rfgj54f4zy3w9j2byxc3-libwebp-1.3.0 /nix/store/4m9xmdvxphc2mh3dgi8akgv3ks64lbql-libtiff-4.5.0-dev /nix/store/f5sqw65lhsl2lm2sp7lylg46idw8p2a5-libavif-0.11.1 /nix/store/y9qa90cg5cy75y08wji8vfn753x0xjya-libXpm-3.5.15-dev",
      "builder": "/nix/store/h6rxqqi4m0vipgvv9czpmwvifdjawb14-bash-5.2-p15/bin/bash",
      "cmakeFlags": "",
      "configureFlags": "--enable-gd-formats",
      "depsBuildBuild": "",
      "depsBuildBuildPropagated": "",
      "depsBuildTarget": "",
      "depsBuildTargetPropagated": "",
      "depsHostHost": "",
      "depsHostHostPropagated": "",
      "depsTargetTarget": "",
      "depsTargetTargetPropagated": "",
      "dev": "/nix/store/lq0yaxn8h80lcmk0spszh02dh4y3zig5-gd-2.3.3-dev",
      "doCheck": "",
      "doInstallCheck": "",
      "enableParallelBuilding": "1",
      "enableParallelChecking": "1",
      "enableParallelInstalling": "1",
      "hardeningDisable": "format",
      "mesonFlags": "",
      "name": "gd-2.3.3",
      "nativeBuildInputs": "/nix/store/mhcirvz2zj0argcd7g5xj7fxxmrgq8b6-autoconf-2.71 /nix/store/97pgbvmsf4j6a7kkvh21xklmairra3c1-automake-1.15.1 /nix/store/s60i3ddb312plbs83r8j7ibivvj5alsc-pkg-config-wrapper-0.29.2",
      "out": "/nix/store/0jpr4lhklp7w5gif5h09mvq18yc3p44d-gd-2.3.3",
      "outputs": "bin dev out",
      "patches": "/nix/store/g6pl8jns2x3ji3rw20cks599nfwljscl-restore-GD_FLIP.patch",
      "pname": "gd",
      "postFixup": "moveToOutput \"bin/gdlib-config\" $dev\n",
      "propagatedBuildInputs": "",
      "propagatedNativeBuildInputs": "",
      "src": "/nix/store/lqis2n1qb9xif50cw7wwcxhibas82l22-libgd-2.3.3.tar.xz",
      "stdenv": "/nix/store/0swa08bbslxc9wbfixbcd196aa1117lx-stdenv-linux",
      "strictDeps": "",
      "system": "x86_64-linux",
      "version": "2.3.3"
    },
    "inputDrvs": {
      "/nix/store/296c13dsdk314yrdk46jcad946p4pahm-libjpeg-turbo-2.1.5.1.drv": [
        "dev"
      ],
      "/nix/store/2qlw2wm333pjn7iaqlvj9ksbwq8ibj86-automake-1.15.1.drv": [
        "out"
      ],
      "/nix/store/6s9l92aczmdsqrvs4jbjf4c1dj23ljm5-bash-5.2-p15.drv": [
        "out"
      ],
      "/nix/store/7ixkn465wh3hari950y1kjfzq9bs6dmx-stdenv-linux.drv": [
        "out"
      ],
      "/nix/store/8ff683l4v63ijik2j6f6ps21r8h0d9fi-libwebp-1.3.0.drv": [
        "out"
      ],
      "/nix/store/bhqynph4acis7my5a6jxb63m7ymqhcxh-libgd-2.3.3.tar.xz.drv": [
        "out"
      ],
      "/nix/store/by6v49mcgavyc04nsfg9c6lgbm2j6smz-pkg-config-wrapper-0.29.2.drv": [
        "out"
      ],
      "/nix/store/cvznzpy5v9xbpsi7cyr507s9pcljpxff-freetype-2.13.0.drv": [
        "dev"
      ],
      "/nix/store/iccm2qjxrz3hwxwmkms0p2kamsqhpwb1-restore-GD_FLIP.patch.drv": [
        "out"
      ],
      "/nix/store/jvdp3xbsgn7xr01dp1kgwn0qky2bzddq-libtiff-4.5.0.drv": [
        "dev"
      ],
      "/nix/store/l7kmgs7cf2ffgi4vyracl2pvhf256wk9-zlib-1.2.13.drv": [
        "dev"
      ],
      "/nix/store/pp3s6xm5mp717d1gv4yx2i23cnhsccli-libavif-0.11.1.drv": [
        "out"
      ],
      "/nix/store/pzw66r9d0s2jzfd1vpkzhq7wqbslvnx7-libXpm-3.5.15.drv": [
        "dev"
      ],
      "/nix/store/rpnr83q06n84zij66c010pkaw31q7pqf-libpng-apng-1.6.39.drv": [
        "dev"
      ],
      "/nix/store/wpzdk53hlqgma92kh7ggrvrakqdw11kp-autoconf-2.71.drv": [
        "out"
      ],
      "/nix/store/xl8xzy22bil7rqhxy6121hpxknyzav49-fontconfig-2.14.0.drv": [
        "dev"
      ]
    },
    "inputSrcs": [
      "/nix/store/6xg259477c90a229xwmb53pdfkn6ig3g-default-builder.sh"
    ],
    "outputs": {
      "bin": {
        "path": "/nix/store/gxwps8pxk904k76syj15fjal7gppymar-gd-2.3.3-bin"
      },
      "dev": {
        "path": "/nix/store/lq0yaxn8h80lcmk0spszh02dh4y3zig5-gd-2.3.3-dev"
      },
      "out": {
        "path": "/nix/store/0jpr4lhklp7w5gif5h09mvq18yc3p44d-gd-2.3.3"
      }
    },
    "system": "x86_64-linux"
  }
}
```

其有三个outpath:bin dev out,gd的三个bin与dev是从out中拷贝的嘛？ =>

其实并不一样

```sh
➜  lq0yaxn8h80lcmk0spszh02dh4y3zig5-gd-2.3.3-dev ls
include  lib  nix-support
➜  lq0yaxn8h80lcmk0spszh02dh4y3zig5-gd-2.3.3-dev cd include
➜  include ls
gd.h  gd_color_map.h  gd_errors.h  gd_io.h  gdcache.h  gdfontg.h  gdfontl.h  gdfontmb.h  gdfonts.h  gdfontt.h  gdfx.h  gdpp.h
➜  include cd ..
➜  lq0yaxn8h80lcmk0spszh02dh4y3zig5-gd-2.3.3-dev ls
include  lib  nix-support
➜  lq0yaxn8h80lcmk0spszh02dh4y3zig5-gd-2.3.3-dev cd lib
➜  lib ls
pkgconfig
➜  lib cd ..
➜  lq0yaxn8h80lcmk0spszh02dh4y3zig5-gd-2.3.3-dev /nix/store/lq0yaxn8h80lcmk0spszh02dh4y3zig5-gd-2.3.3-de
➜  lq0yaxn8h80lcmk0spszh02dh4y3zig5-gd-2.3.3-dev cd /nix/store/0jpr4lhklp7w5gif5h09mvq18yc3p44d-gd-2.3.3
➜  0jpr4lhklp7w5gif5h09mvq18yc3p44d-gd-2.3.3 ;s
zsh: command not found: s
➜  0jpr4lhklp7w5gif5h09mvq18yc3p44d-gd-2.3.3 ls
lib
➜  0jpr4lhklp7w5gif5h09mvq18yc3p44d-gd-2.3.3 cd lib
➜  lib ls
libgd.la  libgd.so  libgd.so.3  libgd.so.3.0.11
```

## 如何形成一个库？

- 找一个目录在下面创建一个default.nix文件，当我们import这个目录而不细指下面的文件时其实就是引用此目录下的default.nix
    
    ```nix
    {
      hello = import ./hello.nix;
      graphviz = import ./graphviz.nix;
    }
    ```
    
    这个nix表达式是一个set：drv名与值的键值对
    
- 在repl中加载两个drv变量
    
    ```sh
      $ nix repl
    nix-repl> :l default.nix
    Added 2 variables.
    nix-repl> hello
    «derivation /nix/store/dkib02g54fpdqgpskswgp6m7bd7mgx89-hello.drv»
    nix-repl> graphviz
    «derivation /nix/store/zqv520v9mk13is0w980c91z7q1vkhhil-graphviz.drv»
    ```
    

### 如何安装自己写的nix表达式？

```sh
$ nix-env -f . -iA graphviz
[...]
$ dot -V
```

-f:指定使用的表达式 -i：安装 -A: 类似:nix-build

## 更高级的input pattern

以上的操作存在一些问题：

- 我们都直接依赖nixpkgs,将import \<nixpkgs>写死在了graphviz与hello的表达式里
- 我们的依赖也是写死的，如在graphviz中的gd库，假如我们想要一个变体没有gd的支持呢
- 我们也没有指定graphviz所依赖的库的具体版本，这样不方便我我们测试

### 让input pattern抽象出来更灵活

- 🔗By <- [没有callPackage时的痛点](http://localhost:1313/posts/niximportpackage/#460DE15A-A2F1-4E11-9D11-D122001FC0C4)
- 原本单个包的表达式都是写死的，我们在default.nix仅是import了，而现在我们要将，包的表达式变为接受参数的函数，这些函数由default.nix传入，已建造不同的包
- 将graphvz.nix变为如下

```nix
  { mkDerivation, lib, gdSupport ? true, gd, pkg-config }:

mkDerivation {
  name = "graphviz";
  src = ./graphviz-2.49.3.tar.gz;
  buildInputs =
    if gdSupport
      then [
          pkg-config
          (lib.getLib gd)
          (lib.getDev gd)
        ]
      else [];
}
```

- 将defult.nix变为以下格式
    
    ```nix
      let
      pkgs = import \<nixpkgs\> {\};
      mkDerivation = import ./autotools.nix pkgs;
    in with pkgs; {
      hello = import ./hello.nix { inherit mkDerivation; };
      graphviz = import ./graphviz.nix { inherit mkDerivation lib gd pkg-config; };
      graphvizCore = import ./graphviz.nix {
        inherit mkDerivation lib gd pkg-config;
        gdSupport = false;
      };
    }
    ```
    
    - 我们可以看到graphviz与grpahvizCore都复用了相同的 graphviz.nix,这样我们就让graphviz抽像了出来
    - \<nixpkgs>也从grphaviz.nix剥离而是作为一个参数由default定义，这样在后续制作包变体时更方便更改
    - 通过在graphviz.nix中加入的条件判断使得库依赖变为可选

### default.nix的本质是什么？

注意它不是函数，不是其他的东西，它是一个set，一个包含了drv的名与值的set



