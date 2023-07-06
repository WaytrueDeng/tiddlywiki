---
title: nixRepl
publish: true
---

## nix repl

### 如何在nix repl中加载其他的nix expression and bring it in to scope?

#### 用:l

```sh
nix-repl> :l <nixpkgs>
Added 19287 variables.
```

#### \<nixpkgs>这个块代表什么意思？

- 🔗By <- [path也可以用&lt;&gt;包裹，其含义是将在NIX\_PATH这个环境变量中搜寻被包裹的文件或者目录名称](http://localhost:1313/posts/nixlanguage/#68de418b-4611-4cbe-9120-400d03605617)

##### 演示

```sh
nix-repl> \<nixpkgs>
/home/waytrue/.nix-defexpr/channels/nixpkgs

➜  nixpkgs ls
CONTRIBUTING.md  lib
COPYING          maintainers
README.md        nixos
default.nix      pkgs
doc              svn-revision
flake.nix
```



