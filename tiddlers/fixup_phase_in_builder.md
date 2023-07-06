---
title: fixup phase in builder
publish: true
---

## 在builder.sh中添加如下语句

```sh
find $out -type f -exec patchelf --shrink-rpath '{}' \; -exec strip '{}' \; 2>/dev/null
```

- 将会用find工具找到所有文件并且执行patchelf与strip命令
- 记得在baseInputs中添加findutils和patchelf作为依赖

## 重建并执行检查运行时依赖

```sh
$ nix-build hello.nix
[...]
$ nix-store -q --references result
/nix/store/94n64qy99ja0vgbkf675nyk39g9b978n-glibc-2.19
/nix/store/md4a3zv0ipqzsybhjb8ndjhhga1dj88x-hello

$ ldd result/bin/hello
 linux-vdso.so.1 (0x00007fff11294000)
 libc.so.6 \=> /nix/store/94n64qy99ja0vgbkf675nyk39g9b978n-glibc-2.19/lib/libc.so.6 (0x00007f7ab7362000)
 /nix/store/94n64qy99ja0vgbkf675nyk39g9b978n-glibc-2.19/lib/ld-linux-x86-64.so.2 (0x00007f7ab770f000)
```



