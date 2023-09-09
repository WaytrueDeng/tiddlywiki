
##  基础软件包管理
  - 如何理解nix-channel的作用？

   像是其他包管理器中的源一样~，算是软件包的合集，添加它

```sh
 $ nix-channel --add https://nixos.org/channels/nixpkgs-unstable
 $ nix-channel --update
 this derivation will be built:
   /nix/store/gkx3a35wvfym0nzw72p42dg73ir0s0pd-home-manager.drv
 building '/nix/store/gkx3a35wvfym0nzw72p42dg73ir0s0pd-home-manager.drv'...
 unpacking channels...
 $ nix-channel --remove nixpkgs #卸载channel
 $ nix-channel --update nixpkgs #更新channel
 ```

  - nixos channel与nix-pkg channel的区别？

    nixos（即完全由nix包管理器构建的系统）中是自动订阅nixos-channel的

    nixos-channel与nix-pkg channel 内容是一样的，除了nixos-channel只包含linux binaries

  - nix-env -qaP 各参数的含义？
    
    -q: –query 查询

```sh
$ nix-env -q
alacritty-0.11.0
home-manager-path
```
-a: –available 如果不加它只会询问本地已下载到/nix/store 里的包

 -P: –attr-path

```sh
$ nix-env -qaP alacritty
nixpkgs.alacritty  alacritty-0.11.0
```

> -P prints the attribute paths that can be used to unambiguously select a package for installation — nix mannul

 其中 nixpkgs.alacritty 即是attr-path其是唯一的 而alacritty-0.11.0 如果你安装了多个channel的时候其可能会有重复

  - nix-env -qa –status 结果的意义？

```sh
$ nix-env -qa --status alacritty
IPS  alacritty-0.11.0
```

 I指已经安装到了用户的environment P:安装在了系统的environment S 指在nix官方维护的二进制源中有相应的缓存（cache)

### 如何使用nix包管理器安装软件？


```sh
$ nix-env --install --attr
$ nix-env -iA
```


> When you ask Nix to install a package, it will first try to get it in pre-compiled form from a binary cache. By default, Nix will use the binary cache [https://cache.nixos.org](https://cache.nixos.org); it contains binaries for most packages in Nixpkgs. Only if no binary is available in the binary cache, Nix will build the package from source. So if nix-env -iA nixpkgs.subversion results in Nix building stuff from source, then either the package is not built for your platform by the Nixpkgs build servers, or your version of Nixpkgs is too old or too new. For instance, if you have a very recent checkout of Nixpkgs, then the Nixpkgs build servers may not have had a chance to build everything and upload the resulting binaries to [https://cache.nixos.org](https://cache.nixos.org)

> The Nixpkgs channel is only updated after all binaries have been uploaded to the cache, so if you stick to the Nixpkgs channel (rather than using a Git checkout of the Nixpkgs tree), you will get binaries for most packages.

-  -A = –attr 可以与上文中的-P 查询配合使用用于选择唯一的软件包


```sh
$ nix-env --uninstall
$ nix-env -e
```

注意卸载不应使用attr名，应使用deriviation名，如应使用alacritty而非nixpkg.alacritty


```sh
$ nix-env -u or (--upgrade)
```

-u 升级软件包





