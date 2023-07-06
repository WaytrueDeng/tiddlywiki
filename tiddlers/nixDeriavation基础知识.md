---
title: nixDeriavation基础知识
publish: true
---

## 什么是derivation?

derivation就像是一种描述文件记录了如何造出一个软件包。如果一个derivation不变 那么总是造出完全一样的软件包

## 如何造出一个derivation?

用nix builtin function(内置函数）, derivation就行

## derivation function所接受参数格式？

### 接受一个参数，这个参数是set类型的

### set中包含了几个属性（attributes)\

#### name: 这个derivation的名字

#### system: 这个derivation 适合在何种CPU架构上的系统上构建

#### builder:构建这个derivation的二进制程序

## 如何查看当前系统的架构？

### nix内建函数 builtins.currentSystem

### 演示

```sh
nix-repl> builtins.currentSystem
"x86_64-linux"

nix-repl>
```

## 当我们在函数中定义一个derivation 然后执行这个函数，这个derivation会被build吗？

### 不会

```sh
nix-repl> d = derivation { name = "myname"; builder = "mybuilder"; system = "mysystem"; }

nix-repl> d
«derivation /nix/store/z3hhlxbckx4g3n9sw91nnvlkjvyw754p-myname.drv»
```

### 当你定义了一个derivation然后执行了它，nix会在nix store下创建一个相应的drv文件

## drv 文件的结构

### 可以通过nix show-derivation 来更有格式的查看drv文件的内容，当然你也可以直接cat

#### 演示

```sh
$ nix show-derivation /nix/store/z3hhlxbckx4g3n9sw91nnvlkjvyw754p-myname.drv
{
  "/nix/store/z3hhlxbckx4g3n9sw91nnvlkjvyw754p-myname.drv": {
    "outputs": {
      "out": {
        "path": "/nix/store/40s0qmrfb45vlh6610rk29ym318dswdr-myname"
      }
    },
    "inputSrcs": [],
    "inputDrvs": {},
    "platform": "mysystem",
    "builder": "mybuilder",
    "args": [],
    "env": {
      "builder": "mybuilder",
      "name": "myname",
      "out": "/nix/store/40s0qmrfb45vlh6610rk29ym318dswdr-myname",
      "system": "mysystem"
    }
  }
}
```

#### 当你执行nix show-derivation时会报错因为还是试验性功能尚未默认启用

```sh
➜  ~ nix show-derivation /nix/store/z3hhlxbckx4g3n9sw91nnvlkjvyw754p-myname.drv
error: experimental Nix feature 'nix-command' is disabled; use '--extra-experimental-features nix-command' to override
```


### 在out项中的path的含义是什么？

#### nix 根据drv文件的特点计算出hash值然后根据独一无二的值赋给产出包的路径

#### TODO nix也有根据产出文件的特性来计算hash值的例如tarball (此处我不太清楚等以后再说)

#### nix可以有多个产出的path，本例中的out只是其中只有一个path的情况

### input derivations 是什么？

#### 我理解为这个drv所依赖的其他的drv，此drv中我们没有依赖任何其他的drv所以为空

### 建造一个drv所依赖的最少的信息包括？

#### dependence 依赖

#### builder 执行建造过程的程序

#### environment variable建造过程中使用的环境变量

#### output path 产出的包所需要存放的路径

## 如上所述执行drv并不会build它，那我们如何真正build一个drv呢？

### 方法一：用nix repl 中的:b

```sh
nix-repl> d = derivation { name = "myname"; builder = "mybuilder"; system = "mysystem"; }

nix-repl> :b d
error: a 'mysystem' with features {} is required to build '/nix/store/z3hhlxbckx4g3n9sw91nnvlkjvyw754p-myname.drv', but I am a 'x86_64-linux' with features {benchmark, big-parallel, kvm, nixos-test, uid-range}
```

### 方法二：用nix store realise

```sh
$ nix-store -r /nix/store/z3hhlxbckx4g3n9sw91nnvlkjvyw754p-myname.drv
➜  nix git:(main) ✗ nix-store -r /nix/store/z3hhlxbckx4g3n9sw91nnvlkjvyw754p-myname.drv
this derivation will be built:
  /nix/store/z3hhlxbckx4g3n9sw91nnvlkjvyw754p-myname.drv
error: a 'mysystem' with features {} is required to build '/nix/store/z3hhlxbckx4g3n9sw91nnvlkjvyw754p-myname.drv', but I am a 'x86_64-linux' with features {benchmark, big-parallel, kvm, nixos-test, uid-range}
```

## 在nix语言中中的derivation类型到底是什么即derivation这个内置的函数它的返回值类型到底是什么？

### 它是attributes set

```sh
nix-repl> builtins.isAttrs d
true
```

### nix中derivation的set中包含哪些键（attributes? 反正是键值对随你怎么叫)

#### 演示

```sh
nix-repl> builtins.attrNames d
[ "all" "builder" "drvAttrs" "drvPath" "name" "out" "outPath" "outputName" "system" "type" ]
```

## 如何在drv中引用其他的drv作为依赖？

### 我们引用的是依赖的outPath而不是依赖drv文件的path

#### 例如我们引用coreutils作为builder,注意当我们每次修改derivation时，产出路径的hash值都会改变

```sh
nix-repl> :l <nixpkgs>
nix-repl> d = derivation { name = "myname"; builder = "${coreutils}/bin/true"; system = builtins.currentSystem; }
nix-repl> :b d
builder for `/nix/store/qyfrcd53wmc0v22ymhhd5r6sz5xmdc8a-myname.drv' failed to produce output path `/nix/store/ly2k1vswbfmswr33hw0kf0ccilrpisnk-myname'
```

##### 在out项中的path的含义是什么？

### 如何快速得到依赖drv的output的path?

#### 你可以使用内置的变为字符串的函数

builtin.toString function:注意这个function它识别的是set中的名为outPath的键的值，所以如果一个set中不包括这个键的话将会报错

```sh
nix-repl> builtins.toString d
"/nix/store/40s0qmrfb45vlh6610rk29ym318dswdr-myname"

nix-repl> builtins.toString d
"/nix/store/40s0qmrfb45vlh6610rk29ym318dswdr-myname"

nix-repl> builtins.toString { a = "b"; }
error: cannot coerce a set to a string

       at «string»:1:1:

            1| builtins.toString { a = "b"; }
             | ^
```

#### 或者你可以使用interpolate string的方式

- 🔗By <- [被引用：](http://localhost:1313/posts/nixlanguage/#0a7e7d8e-ec6b-4b14-be49-5a58b709c311)

“${d}”

```sh
nix-repl> "${d}"
"/nix/store/40s0qmrfb45vlh6610rk29ym318dswdr-myname"
```

### 引入依赖后的derivation看起来如何？

#### 演示

```sh
{
  "/nix/store/bsdky939pivf2j4v55blcnacllrv0mz9-myname.drv": {
    "args": [],
    "builder": "/nix/store/l8g5py3i39sq8afzi9vfmpw5igbqs84r-coreutils-9.1/bin/true",
    "env": {
      "builder": "/nix/store/l8g5py3i39sq8afzi9vfmpw5igbqs84r-coreutils-9.1/bin/true",
      "name": "myname",
      "out": "/nix/store/ndxgmy6ckwlvpk0dcnywg26ksz33a027-myname",
      "system": "x86_64-linux"
    },
    "inputDrvs": {
      "/nix/store/y235v15p96w6xpxkkd207q06vvpsilra-coreutils-9.1.drv": [
        "out"
      ]
    },
    "inputSrcs": [],
    "outputs": {
      "out": {
        "path": "/nix/store/ndxgmy6ckwlvpk0dcnywg26ksz33a027-myname"
      }
    },
    "system": "x86_64-linux"
  }
}
```

#### TODO nix是如何识别drv中的依赖的？根据drv文件中是否有指向其他drv文件的path吗？还是根据${} string interpolated 行为 或者是builtin.toString的这个函数调用的行为？

## 使用bash script作为建造包的建造者

### 在nix使用bash时不使用 hash bangs以及不使用/usr/bin/env的原因？

#### 不使用hash相关的path是因为在写此script时我们尚不知道产品的hash 值开头的path

#### 不使用/usr/bin/env的原因是因为PATH变量被清空了

##### /usr/bin/env程序的作用是什么？

###### 设置环境变量中的键值然后运行相关的命令

### 在nix building过程中PATH的如何变化？

#### 将会被清空使得建造过程中纯净

### 我们如何将builder所依照的script传给derivation?

#### 如上所述内置的derivation函数的第一个参数是一个set，这个set的内容如上述, set中可以设置一args的attribute 其为被用作参数传给derivation

### 我们首先写一个基础的script

```sh
➜  ~ bat builder.sh
───────┬──────────────────────────────────────────────────────
       │ File: builder.sh
───────┼──────────────────────────────────────────────────────
   1   │ declare -xp
   2   │ echo foo > $out
───────┴──────────────────────────────────────────────────────
➜  ~
```

#### 在bash中declare -xp的作用是什么？

##### 是为了方便我们看 build过程中的环境变量是怎么样的

###### 为什么我们不用/usr/bin/env来查看环境变量呢？

- 因为env程序属于coreutils，而我们现在没有设置coreutils作为依赖,注意bash是一个单独的包而不是coreutils所提供的
    
    ```sh
    nix-repl> "${bash}"
    "/nix/store/h6rxqqi4m0vipgvv9czpmwvifdjawb14-bash-5.2-p15"
    ```
    

##### 演示

```sh
declare: declare [-aAfFgiIlnrtux] [-p] [name[=value] ...]
    Set variable values and attributes.

    Declare variables and give them attributes.  If no NAMEs are given,
    display the attributes and values of all variables.

    Options:
      -f	restrict action or display to function names and definitions
      -F	restrict display to function names only (plus line number and
                source file when debugging)
      -g	create global variables when used in a shell function; otherwise
                ignored
      -I	if creating a local variable, inherit the attributes and value
                of a variable with the same name at a previous scope
      -p	display the attributes and value of each NAME

    Options which set attributes:
      -a	to make NAMEs indexed arrays (if supported)
      -A	to make NAMEs associative arrays (if supported)
      -i	to make NAMEs have the `integer' attribute
      -l	to convert the value of each NAME to lower case on assignment
      -n	make NAME a reference to the variable named by its value
      -r	to make NAMEs readonly
      -t	to make NAMEs have the `trace' attribute
      -u	to convert the value of each NAME to upper case on assignment
      -x	to make NAMEs export

    Using `+' instead of `-' turns off the given attribute.

    Variables with the integer attribute have arithmetic evaluation (see
    the `let' command) performed when the variable is assigned a value.

    When used in a function, `declare' makes NAMEs local, as with the `local'
    command.  The `-g' option suppresses this behavior.

    Exit Status:
    Returns success unless an invalid option is supplied or a variable
    assignment error occurs.
```

#### 在nix builder script中的$out变量指的是？


##### 根据nix drv所记算出来的out put path 将在build过程中作为环境变量传递给builder

### 写一个简单的drv

```sh
nix-repl>  d = derivation { name = "foo"; builder = "${bash}/bin/bash"; args = [ ./builder.sh ]; system = builtins.currentSystem; }

nix-repl> :b d

This derivation produced the following outputs:
  out -> /nix/store/74qb5z7id3sqn09ql6p4xl2j2gylkpav-foo
```

#### 这个drv的产出是一个文件，文件的内容是foo,就如同我们在script中写的那样 ``echo foo > $out``

#### 为什么args中的script路径不加"""?

##### 因为这样会被nix视为当前的路径，如果加了"“后就会被原样当作bash的参数,这时bash所理解的路径其实就成了在建造这个包的过程中所在的临时路径，所以就找不到builder.sh

#### nix在执行drv时所得出的out put path中的hash值是否会随着drv中的bash script的内容的变化而变化？

##### 我们将bash script的内容改为

```sh
➜  ~ cat builder.sh
declare -xp
echo foo > $out
echo foo > $out
```

发现的确drv文件与output的path都发生了变化

```sh
nix-repl> :b d

This derivation produced the following outputs:
  out -> /nix/store/d2bnd9q94n9a1gc7irx7mlilc43chyjy-foo
```

### 如何读出在nix建造过程中builder的标准输出？

#### 用nix-store –read-log [outputPath]

```sh
declare -x HOME="/homeless-shelter"
declare -x NIX_BUILD_CORES="8"
declare -x NIX_BUILD_TOP="/build"
declare -x NIX_LOG_FD="2"
declare -x NIX_STORE="/nix/store"
declare -x OLDPWD
declare -x PATH="/path-not-set"
declare -x PWD="/build"
declare -x SHLVL="1"
declare -x TEMP="/build"
declare -x TEMPDIR="/build"
declare -x TERM="xterm-256color"
declare -x TMP="/build"
declare -x TMPDIR="/build"
declare -x builder="/nix/store/h6rxqqi4m0vipgvv9czpmwvifdjawb14-bash-5.2-p15/bin/bash"
declare -x name="foo"
declare -x out="/nix/store/74qb5z7id3sqn09ql6p4xl2j2gylkpav-foo"
declare -x system="x86_64-linux"
```

### nix造包过程中的环境变量探究


#### 演示

```sh
declare -x HOME="/homeless-shelter"
declare -x NIX_BUILD_CORES="8"
declare -x NIX_BUILD_TOP="/build"
declare -x NIX_LOG_FD="2"
declare -x NIX_STORE="/nix/store"
declare -x OLDPWD
declare -x PATH="/path-not-set"
declare -x PWD="/build"
declare -x SHLVL="1"
declare -x TEMP="/build"
declare -x TEMPDIR="/build"
declare -x TERM="xterm-256color"
declare -x TMP="/build"
declare -x TMPDIR="/build"
declare -x builder="/nix/store/h6rxqqi4m0vipgvv9czpmwvifdjawb14-bash-5.2-p15/bin/bash"
declare -x name="foo"
declare -x out="/nix/store/74qb5z7id3sqn09ql6p4xl2j2gylkpav-foo"
declare -x system="x86_64-linux"
```

#### nix $HOME 与 $PATH变量在建造过程中被设置为了不存在的路径目的是？

##### 使得建造过程不依赖此二者$

#### CORES和STORE的路径可以通过nix 设置

#### PWD与TMP等变量可以清楚的看出是在一个临时的路径中完成的建造

#### 可以看到out作为变量传给了建造过程中

#### $out可以类比为make中的–prefix而部DESTDIR

##### make中的–prefix与DESTDIR的作用是？

###### 演示

> `./configure --prefix=***`

###### **Number 1** determines where the package will go when

it is installed, and where it will look for its associated files when it is run. It’s what you should use if you’re just compiling something for use on a single host.

###### —–

###### 演示

> `make install DESTDIR=***`

**Number 2** is for installing to a temporary directory which is not where the package will be run from. For example this is used when building `deb` packages. The person building the package doesn’t actually install everything into its final place on his own system. He may have a different version installed already and not want to disturb it, or he may not even be root. So he uses

###### 演示

```nil
./configure --prefix=/usr
```

###### so the program will expect to be installed in `/usr` when it runs, then

###### 演示

```nil
make install DESTDIR=debian/tmp
```

###### to actually create the directory structure.

###### —–

###### 演示

> make install prefix=***

**Number 3** is going to install it to a different place but not create all the directories as `DESTDIR=/foo/bar/baz` would. It’s commonly used with GNU stow via

###### 演示

```nil
./configure --prefix=/usr/local && make && sudo make install prefix=/usr/local/stow/foo
```

###### , which would install binaries in `/usr/local/stow/foo/bin`. By comparison,

###### 演示

```nil
make install DESTDIR=/usr/local/stow/foo
```

###### would install binaries in `/usr/local/stow/foo/usr/local/bin`.

### 新的drv文件类容可以看出args中的build.sh已经被nix复制到了nix store的统一目录下，其目的是防止在建造包的过程中被更改

```sh
7.4. The .drv contents

We added something else to the derivation this time: the args attribute. Let's see how this changed the .drv compared to the previous pill:

$ nix show-derivation /nix/store/i76pr1cz0za3i9r6xq518bqqvd2raspw-foo.drv
{
  "/nix/store/i76pr1cz0za3i9r6xq518bqqvd2raspw-foo.drv": {
    "outputs": {
      "out": {
        "path": "/nix/store/gczb4qrag22harvv693wwnflqy7lx5pb-foo"
      }
    },
    "inputSrcs": [
      "/nix/store/lb0n38r2b20r8rl1k45a7s4pj6ny22f7-builder.sh"
    ],
    "inputDrvs": {
      "/nix/store/hcgwbx42mcxr7ksnv0i1fg7kw6jvxshb-bash-4.4-p19.drv": [
        "out"
      ]
    },
    "platform": "x86_64-linux",
    "builder": "/nix/store/q1g0rl8zfmz7r371fp5p42p4acmv297d-bash-4.4-p19/bin/bash",
    "args": [
      "/nix/store/lb0n38r2b20r8rl1k45a7s4pj6ny22f7-builder.sh"
    ],
    "env": {
      "builder": "/nix/store/q1g0rl8zfmz7r371fp5p42p4acmv297d-bash-4.4-p19/bin/bash",
      "name": "foo",
      "out": "/nix/store/gczb4qrag22harvv693wwnflqy7lx5pb-foo",
      "system": "x86_64-linux"
    }
  }
}
```

## 打包一个C语言程序

### C程序的源码如下

```c
void main() {
  puts("Simple!");
}
```

### 添加一个builder script:

```sh
➜  workround bat simple_builder.sh
───────┬──────────────────────────────────────────────────────
       │ File: simple_builder.sh
───────┼──────────────────────────────────────────────────────
   1   │ export PATH="$coreutils/bin:$gcc/bin"
   2   │ mkdir $out
   3   │ gcc -o $out/simple $src
───────┴──────────────────────────────────────────────────────
➜  workround
```

#### 在这个script中的 $src变量的值是？

### 建造这个包

```sh
nix-repl> :l <nixpkgs>
Added 19287 variables.

nix-repl> simple = derivation { name = "simple"; builder = "${bash}/bin/bash"; args = [ ./simple_builder.sh ]; gcc = gcc; coreutils = coreutils; src = ./simple.c; system = builtins.currentSystem; }
nix-repl> :b simple

This derivation produced the following outputs:
  out -> /nix/store/zz1p7gar61hcbgjaycnp2y3g3z8fmlmb-simple
```

#### 在这个derivation的定义过程中，gcc=gcc,builder=buider像以前学过的哪个关键词的作用？

##### gcc左侧是在simple drv中的name, 而右侧则是nixpkgs中的gcc drv



#### TODO gcc=gcc这种结构是如何传递给建造过程的？左侧的gcc变量是如何被建造脚本引用的？

##### 添加一个builder script

在这个脚本中作为PATH变量时string interpolating

#### src attr的作用是什么？

##### 没有什么特别的作用，就像args一样的, 后面的path将会被nix 复制到相应的nix store并将新的路径赋给变量src

#### 可以看出在传递给内建函数derivation的set中的每一个attr与 building过程中的关系是？

##### 它们都会被转变为string类型然后作为一个环境变量传递给builder

******

## 受够了nix repl?

### 创建一个simple.nix的文件

```nix
let
  pkgs = import <nixpkgs> {};
in
  pkgs.stdenv.mkDerivation {
    name = "simple";
    builder = "${pkgs.bash}/bin/bash";
    args = [ ./simple_builder.sh ];
    gcc = pkgs.gcc;
    coreutils = pkgs.coreutils;
    src = ./simple.c;
    system = builtins.currentSystem;
}
```

#### ~import~的用法是什么？

##### import是nix 提供的内建的function, 用于解析.nix文件的

##### 注意import的时候import的文件的scope是不会被继承给被import文件中的变量的，比如我在import文件中定义了一个变量x=3,但是这个x=3就不会被imported文件中的x所继承 test.nix的内容是：x

```src
nix-repl> let x = 5; in import ./test.nix
error: undefined variable `x' at /home/lethal/test.nix:1:1
```

##### 所以我们不能通过import文件中的变量传参给imported中变量，那么我们要如何才能传递参数给.nix文件呢？

```nix
{ a, b ? 3, trueMsg ? "yes", falseMsg ? "no" }:
if a > b
  then builtins.trace trueMsg true
  else builtins.trace falseMsg false
```

```sh
nix-repl> import ./test.nix { a = 5; trueMsg = "ok"; }
trace: ok
true
```

就是我们在nix文件中写成函数的形式，然后再在import文件中调用这个函数，在调用的过程中将变量值传给它，怎么理解呢就是我们通过直接引用它的函数然后直接传递给它相应的值而不是利用已有的外界scope中的值

#### ~import \<nixpkgs> {};~后面的{}的意义是什么？

##### import \<nixpkgs>实际上是evaluate了一个function,这个function的返回值是一个set of derivation

##### 所以我们不能通过import文件中的变量传参给imported中变量，那么我们要如何才能传递参数给.nix文件呢？]

经过这个例子我们值道import了函数后接的set之类的为其参数，相当于引用后又调用使之执行



##### [](https://nixos.org/guides/nix-pills/nixpkgs-parameters.html#idm140737319682512)

###### 这个{}为参数，写在在nixpkgs中的而不是nix中，nixpkgs看 它是不是空的，如何是空的就会读取~/.config/nixpkgs/config.nix中的内容

*********

#### nix repl中的derivation function变成了 `pkgs.stdenv.mkDerivation`

#### 如何构建一个simple.nix?

##### `nix-build simple.nix`

#### nix-build所产生的文件是什么？

##### 它会在当前的目录创建一个result目录，里面是symlink指向nix store中的out path

```sh
➜  workround exa -aT result
result -> /nix/store/m42r8rvq3yb2vv5v06k6fb0nvl170z8g-simple
```

#### nix-build的过程是如何实现的？

##### nix-instantiate 执行nix文件然后根据所解析到的derivation set产生相应的drv文件

##### nix-store -r *.drv realise drv文件

### 经过改版的simple.nix文件

```nix
let
  pkgs = import <nixpkgs> {};
in
  pkgs.stdenv.mkDerivation {
    name = "simple";
    builder = "${pkgs.bash}/bin/bash";
    args = [ ./simple_builder.sh ];
    inherit (pkgs) gcc coreutils;
    src = ./simple.c;
    system = builtins.currentSystem;
}
```

Lastly, inherit (pkgs) gcc coreutils; is equivalent to gcc=pkgs.gcc; coreutils=pkgs.coreutils;.



#### 需要再次注意inherit关键词的用处是仅限于？


##### 仅限set内使用以继承外部环境的变量，外部环境没有要求是set



