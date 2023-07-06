---
title: nixGL
publish: true
---

### nixGL的作用是什么？

#### 当我们在非nixos的linux发行版中使用nix包管理器是无法正常使用gui即图形化界面应用的所以我们需要nixGL其旨在将nix应用包裹起来并使之使用正确的nix环境中的opengl库

### 演示

```sh
$ nix-channel --add https://github.com/guibou/nixGL/archive/main.tar.gz nixgl && nix-channel --update
$ nix-env -iA nixgl.auto.nixGLDefault   # or replace `nixGLDefault` with your desired wrapper
```





