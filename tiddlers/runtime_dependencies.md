---
title: runtime dependencies
publish: true
---

## 什么是runtime dependencies?

- build dependences是建造这个包之前所依赖的软件
- 而runtime:则是包建造好之后如果nix在它的output目录里找到了其他drv的output，那么我们就认为其为相应的runtime dependences

## nix对待buildtime dependencies与runtime dependencies的区别？

- build dependencies会被nix自动识别因为我们会在derivation中声明，而runtime则未被特地指出

## 如何查找nix包的runtime dependences?

- 我高度怀疑是错误的 nix-pills说的是：用nix-store –dump一个drv,然后再它的输出中找到out所包含的对应的drv即是其runtime dependences
    
    ```sh
    ➜  workround nix-store --dump /nix/store/5086c26jdq7xcyan64cp8w8gxyhbzyyh-hello.drv
    nix-archive-1(typeregulacontentsXDerive([("out","/nix/store/58nwz94j7knkgdvhfiv6q678fny4kzg8-hello","","")],[("/nix/store/18h8jz9vj7yb86lqahjq96bzyxg62kff-binutils-2.40.drv",["out"]),("/nix/store/1fqpzw45flwz5i5v4injpqh2fg4h8ij7-gawk-5.2.1.drv",["out"]),("/nix/store/40wj3dkfky4xcn3arv82y5fkdyi9h1nx-gnumake-4.4.1.drv",["out"]),("/nix/store/6s9l92aczmdsqrvs4jbjf4c1dj23ljm5-bash-5.2-p15.drv",["out"]),("/nix/store/dcc9bzivivd2yymvmj78mph1sfiv4nb6-gnugrep-3.11.drv",["out"]),("/nix/store/dn0kwiaw1yzq2i13919r29r0y4xgmn12-gnutar-1.34.drv",["out"]),("/nix/store/kpj5adpxdw639x8m8hwqwh2v41qddxa5-gzip-1.12.drv",["out"]),("/nix/store/p8vh8i416h2mxfvqjp2dxc8138zikzk0-gcc-wrapper-12.2.0.drv",["out"]),("/nix/store/slmc3fgfwq4yhafrd9g355i9klwzvgzx-gnused-4.9.drv",["out"]),("/nix/store/y235v15p96w6xpxkkd207q06vvpsilra-coreutils-9.1.drv",["out"])],["/nix/store/5qsq9zk6a0lkj9j084s2860cxj43mqyl-builder.sh","/nix/store/svc70mmzrlgq42m9acs0prsmci7ksh6h-hello-2.10.tar.gz"],"x86_64-linux","/nix/store/h6rxqqi4m0vipgvv9czpmwvifdjawb14-bash-5.2-p15/bin/bash",["/nix/store/5qsq9zk6a0lkj9j084s2860cxj43mqyl-builder.sh"],[("baseInputs","/nix/store/g1ajd0l91n1ryyw779wlfhf73imbh4cf-gnutar-1.34 /nix/store/4w0m22sx8yw9s1kw35f72cfak2qww8kh-gzip-1.12 /nix/store/914m27x3h5mfbxxdv8n5bcg1hqbl808d-gnumake-4.4.1 /nix/store/c22pksfcw1nnkcxlf5xm7ljxhngg3n65-gcc-wrapper-12.2.0 /nix/store/l8g5py3i39sq8afzi9vfmpw5igbqs84r-coreutils-9.1 /nix/store/rw2r8jqvbsmpq5kmgwvkv9pd833k9h3z-gawk-5.2.1 /nix/store/figk1iqjicv30sa9qnvbbzdb81bzsh1c-gnused-4.9 /nix/store/4wwylhblws9na4ghmvsia38kimxl43g4-gnugrep-3.11 /nix/store/shw0b6wv2xdvyj71b1fj147i83awrqfz-binutils-2.40"),("buildInputs",""),("builder","/nix/store/h6rxqqi4m0vipgvv9czpmwvifdjawb14-bash-5.2-p15/bin/bash"),("name","hello"),("out","/nix/store/58nwz94j7knkgdvhfiv6q678fny4kzg8-hello"),("src","/nix/store/svc70mmzrlgq42m9acs0prsmci7ksh6h-hello-2.10.tar.gz"),("system","x86_64-linux")]))%
    ```
    
- nix-pill操作时用的方法是:
    
    ```sh
    ➜  workround nix-store -q --references /nix/store/58nwz94j7knkgdvhfiv6q678fny4kzg8-hello
      /nix/store/x33pcmpsiimxhip52mwxbb5y77dhmb21-glibc-2.37-8
      /nix/store/325bszyns4mialm4dsqzxnkrh2q3yr04-glibc-2.37-8-dev
      /nix/store/7ls5xhx6kqpjgpg67kdd4pmbkhna4b6c-gcc-12.2.0-lib
      /nix/store/kf147vkmb7a7z17lkpgj2d6y7w75nf7v-gcc-12.2.0
      /nix/store/58nwz94j7knkgdvhfiv6q678fny4kzg8-hello
    ```
    

### 在我们自己造的hello程序中gcc竟然成了依赖这是为什么呢？

```sh
➜  workround  strings result/bin/hello|grep gcc
/nix/store/x33pcmpsiimxhip52mwxbb5y77dhmb21-glibc-2.37-8/lib:/nix/store/7ls5xhx6kqpjgpg67kdd4pmbkhna4b6c-gcc-12.2.0-lib/lib
/nix/store/kf147vkmb7a7z17lkpgj2d6y7w75nf7v-gcc-12.2.0/lib/gcc/x86_64-unknown-linux-gnu/12.2.0/include
```

- 我们通过strings命令发现hello的二进制程序包含了gcc的path
- hello程序中包含的gcc的path 为ld rpath

#### strings命令的功能是什么？

> NAME strings - print the sequences of printable characters in files

也就是说strings会显示文件中的所有内输出为文字的东西，实际上我用vim打开了这个二进值文件发现其中确实有很多文字然后也找到了相应的gcc的path

#### 什么是ld rpath?

- In computing, rpath designates the run-time search path hard-coded in an executable file or library. Dynamic linking loaders use the rpath to find required libraries.
- 在建造过程中将gcc加入了hello的rpath中因为它猜测其在运行时有用，但是实际上没有用

#### 如何精简rpath到真正需要使用的库？

- 用nix作者写的patchelf工具
    
    > This removes from the RPATH all directories that do not contain a library referenced by DT_NEEDED fields of the executable or library. For instance, if an executable references one library libfoo.so, has an RPATH /lib:/usr/lib:/foo/lib, and libfoo.so can only be found in /foo/lib, then the new RPATH will be /foo/lib.
    
- nix pill说即使用了patchelf,gcc库还会在依赖中存在，因为出于一些debugging information的原因
    
- 我们需要使用strip工具
    
    > strip(1) - Linux man page Name
    > 
    > strip - Discard symbols from object files.
    

## 为什么不用担心未明确声明runtime dependences?

> There’s really no black magic involved. It’s something that at first glance makes you think “no, this can’t work in the long term”, but at the same time it works so well that a whole operating system is built on top of this magic.
> 
> In other words, Nix automatically computes all the runtime dependencies of a derivation, and it’s possible thanks to the hash of the store paths.

## 弄清真正的运行时的意义？

能让我们只复制尽可能少的文件到其它的机器上就能让程序完美的跑起来，便于部署



