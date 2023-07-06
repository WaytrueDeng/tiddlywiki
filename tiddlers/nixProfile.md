---
title: nixProfile
publish: true
---

## nix profile?

### 什么是profile

#### profile是指向nix store (_nix/store_)中某个user-env的软链接

#### profile存在于用户的家目录中.nix-profile以及/nix/var/nix-profiles中，其中家目录的profile实际上也是指向后者的软链接，而后者如上所述则继续指向位于nix-store中的某个版本的user-env

#### 拿我本人的机子举例

```sh
➜  ~ exa -al ~/ | grep nix
.rw-r--r--   81 waytrue 13 Feb 12:22 .nix-channels
drwxr-xr-x    - waytrue  9 Jun 20:41 .nix-defexpr
lrwxrwxrwx   46 waytrue 13 Feb 12:25 .nix-profile -> /nix/var/nix/profiles/per-user/waytrue/profile

➜  profiles exa /nix/var/nix/profiles -al
lrwxrwxrwx 14 root 13 Feb 11:54 default -> default-2-link
lrwxrwxrwx 60 root 13 Feb 11:54 default-1-link -> /nix/store/j91avxymcqd38s0ky8p4f2m8yxr5pgj6-user-environment
lrwxrwxrwx 60 root 13 Feb 11:54 default-2-link -> /nix/store/mrpcl5ljv98xrahl19vwm2ixdblgbyy2-user-environment
drwxr-xr-x  - root 13 Feb 11:57 per-user
```

### 引入profile后有什么用呢？

#### 引入profile后可以让每个用户有自己独立的软件管理，比如我在root用户下可以安装foo-1.00,我在我waytrue用户下 则可以安装foo-2.00使得同一台电脑上的不同用户可以使用不同版本的软件 这在传统的linux下是很难实现的

#### profile还有个重要的作用是可以实现版本控制，我每一次安装卸载或升级软件，实际上都是nix将所操作软件的路径给加到上述/nix/store的env文件夹中，但是每次操作都会加上专属的hash作为前缀，所以可以在store中存在不同的env，这样就能将历史的env保存下来，nix再用相应的profile指向相应的env,当回溯版本时，将家目录下的profile指向系统/nix/var/nix/profile中相应历史版本的profile就能实现回退

曾经想过为什么不让家目录下的profile直接指向nix-store而要经过中间的/nix/var/nix/profile中转？ 想了下其实中转的这个可以当作一个历史保存的作用，其有序号记录了每次的generation

### 如何使用profile进行版本回退？

#### 如何查看拥有哪些历史版本？

##### 演示

```sh
$ nix-env --list-generations
   1   2023-02-13 12:25:33
   2   2023-02-13 14:16:35
   3   2023-02-13 14:29:41
   4   2023-02-13 15:24:09
   5   2023-02-13 16:10:14
   6   2023-03-14 16:53:24
   7   2023-05-10 09:05:48
   8   2023-06-09 20:55:16   (current)
```

##### 与之相对应的是我/nix/var/nix/profiles/per-user/waytrue中的profile文件

###### 演示

```sh
waytrue ls -la | grep profile
lrwxrwxrwx 1 waytrue waytrue  14 Jun  9 20:55 profile -> profile-8-link
lrwxrwxrwx 1 waytrue waytrue  60 Feb 13 12:25 profile-1-link -> /nix/store/82n5lgjfw6if4qg3ag7dy56ij1gfhn1w-user-environment
lrwxrwxrwx 1 waytrue waytrue  60 Feb 13 14:16 profile-2-link -> /nix/store/0zmpyaz4r5y69fs299jazz038p1d8l12-user-environment
lrwxrwxrwx 1 waytrue waytrue  60 Feb 13 14:29 profile-3-link -> /nix/store/grf9xii5lk2bwl1zmmj5kz10qg4zkghr-user-environment
lrwxrwxrwx 1 waytrue waytrue  60 Feb 13 15:24 profile-4-link -> /nix/store/8igsjrfvnp2z64fvasfi9kxrfsbhfkbc-user-environment
lrwxrwxrwx 1 waytrue waytrue  60 Feb 13 16:10 profile-5-link -> /nix/store/fg2bli796xnsnndnzjjbair6sixigf1b-user-environment
lrwxrwxrwx 1 waytrue waytrue  60 Mar 14 16:53 profile-6-link -> /nix/store/d2fd4xfyqay1bb7953dk3ygwszkh7c5q-user-environment
lrwxrwxrwx 1 waytrue waytrue  60 May 10 09:05 profile-7-link -> /nix/store/vzap2y9cwpi5mm8fhyamlpj7znlx0qdx-user-environment
lrwxrwxrwx 1 waytrue waytrue  60 Jun  9 20:55 profile-8-link -> /nix/store/yw53vfl574ivir5a0hdf3m4in2cgmwli-user-environment
```

#### 如何切换到对应的版本？

##### 演示

```sh
$ nix-env --switch-generation  序号
```

*****

### 如何切换profile?

#### 可以使用如下命令使得家目录下的profile指向你所特点的/nix~下的profile

```sh
$ nix-env --switch-profile /nix/var/nix/profiles/xxxxxxx

➜  ~ nix-env --switch-profile /nix/var/nix/profiles/default
➜  ~ ls -la .nix-profile
lrwxrwxrwx 1 waytrue waytrue 29 Jun  9 21:54 .nix-profile -> /nix/var/nix/profiles/default
```

### 如何在不切换profile的情况下对特点profile进行操作？

#### 演示

```sh
$ nix-env --profile /nix/var/nix/profiles/other-profile --install --attr nixpkgs.subversion
```

> All nix-env operations work on the profile pointed to by ~/.nix-profile, but you can override this using the –profile option (abbreviation -p):
> 
> This will not change the ~/.nix-profile symlink.



