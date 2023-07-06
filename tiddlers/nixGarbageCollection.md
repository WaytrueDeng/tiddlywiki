---
title: nixGarbageCollection
publish: true
---

## garbage collection

### 什么是以及为什么要garbage collection？

#### 如上所述nix的一大优点就是支持版本回溯，所以历史的版本虽然不在当前的profile中呈现了，但是其并没有从硬盘上（/nix/store)中删除. 当我们使用nix-env -e(即 –uninstall)时我们只是删除了相应的软件在我们env中的软链接，但历史版本越来越多时我们的硬盘将会不负重担，所以我们需要进行垃圾清理以确保不再需要的软件不再在我们硬盘中存在

### 哪些软件会被视为garbage?

#### 如果一个软件仍然被任何一个profile所引用，那么我们是不是就不应该对其进行垃圾处理。所以我们需要删除不需要的profile 使这些不需要的软件解放出来变得孤立，这样它们就会被视为垃圾而被清理

### 如何删除不需要的profile?


#### 我们可以删除除当前profile以外的所有的old profile

```sh
$ nix-env --delete-generations old
```

#### 我们也可以删除特定generation的profile

```sh
$ nix-env --delete-generations 10 11 14
```

#### 我们也可以删除指定时间以前的profile

```sh
$ nix-env --delete-generations 14d
```

### 如何garbage collection?

#### 注意是nix-store命令而不是nix-env命令，因为这是对store文件夹进行操作而不是对profile进行操作

```sh
$nix-store --gc
$nix-store --gc --print-dead(或者改成live) #以显示哪些会被删除或保留
```

#### 有一个简单的工具

```sh
$ nix-collect-garbage -d
```

> There is also a convenient little utility nix-collect-garbage, which when invoked with the -d (–delete-old) switch deletes all old generations of all profiles in /nix/var/nix/profiles

#### 清除手动nix-build的内容

```sh
$ nix-channel --update
$ nix-env -u --always
$ rm /nix/var/nix/gcroots/auto/*
$ nix-collect-garbage -d
```

### 什么是以及作用是:garbage collector roots?

#### 是/nix/var/nix/gcroots目录

#### 在此目录中存放的是指向nix store的软链接

#### 在此目录中软链接所指向的derivation将类似于白名单连同其依赖均不会被垃圾清理

#### 在此目录下的子目录中的软连接也会被扫描



