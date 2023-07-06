---
title: nixImportPackage
publish: true
---

## callPackage的方式的流行程度

- callPackage是当前nixxpkgs中importing package的标准方式

## 构建一个callPackage函数的意义在于？

- 在[[nixCreatRepo]]中我们学会了将单个包的nix表达示抽象出来关注建造环节，而具体的参数通过defaul.nix这个调用者传递

### 没有callPackage时的痛点

- 在单个包的nix表达式中我们常会有如下的表达式
    
    ```nix
      { input1, input2, ... }:
    ...
    ```
    
    这种情况可以在 [让input pattern抽象出来更灵活](http://localhost:1313/posts/nixcreatrepo/#1B96EB4E-A9CA-4EFC-BB2F-0C091C1B02E8) 见到
- caller即default.nix中可以见到同样的情况
    
    ```nix
       rec {
      lib1 = import package1.nix { inherit input1 input2 ...; };
      program2 = import package2.nix { inherit inputX inputY lib1 ...; };
    }
    ```
    
- 像input1,input2这些变量我们在两个文件中都写了相同的，有没有什么方法能让我们只在builder里写这个变量名，然后它就会自动的传递给caller?

### 构建callPackage应具有的功能？

```nix
{
  lib1 = callPackage package1.nix { };
  program2 = callPackage package2.nix { someoverride = overriddenDerivation; };
}
```

- 它需要能import我们指定的package.nix的表达式然后返回一个能接受参数的函数
- 它能够决定我们传入参数的名称，只用在caller中写明,而不用写两次
- 它能够提供一些默认的参数，并且让我们有能力重新覆盖一些参数

## 如何构建一callPackage函数

### 如何获取传入参数的名

我们的目的是只在builder中写一次参数名，所以我们需要有个地方存取它然后自动把它加到caller里

- builtins.functionArgs可以完成这个功能,如下：
    
    ```sh
    nix-repl> add = { a ? 3, b }: a+b
    nix-repl> builtins.functionArgs add
    { a = true; b = false; }
    ```
    
    注意这个函数它的确能返回所给函数的参数名，但是,它不能返回默认的值它返回的true or false指的是这个参数是否有默认值

### 如何传入

```nix
nix-repl> values = { a = 3; b = 5; c = 10; }
nix-repl> builtins.intersectAttrs values (builtins.functionArgs add)
{ a = true; b = false; }
nix-repl> builtins.intersectAttrs (builtins.functionArgs add) values
{ a = 3; b = 5; }
```

#### interSectAttrs的作用是什么？

> intersectAttrs e1 e2
> 
> Return a set consisting of the attributes in the set e2 which have the same name as some attribute in e1.
> 
> Performs in O(n log m) where n is the size of the smaller set and m the larger set’s size.

```sh
nix-repl> set1 = {a1=1;b1=2;c1=3;}

nix-repl> set2 = {a1=1;b1=2;c1=3;c2=3;}
nix-repl> builtins.intersectAttrs set1 set2
{ a1 = 1; b1 = 2; c1 = 3; }
nix-repl> builtins.intersectAttrs set2 set1
{ a1 = 1; b1 = 2; c1 = 3; }
```

- 其实你看set1 set2的顺序并不重要,只是取交集？真的吗？错了 =>

```sh
nix-repl> set2 = {a1=true;b1=true;c1=true;c2=true;}
nix-repl> builtins.intersectAttrs set2 set1
{ a1 = 1; b1 = 2; c1 = 3; }
nix-repl> builtins.intersectAttrs set1 set2
{ a1 = true; b1 = true; c1 = true; }
```

所事实上，它的确是取交集，但是取的是变量名的交集形成新的set，而新的set对应变量的值则是从后者中取出

#### 写一个简单的callPackage函数

```sh
nix-repl> callPackage = set: f: f (builtins.intersectAttrs (builtins.functionArgs f) set)
nix-repl> callPackage values add
8
nix-repl> with values; add { inherit a b; }
8
```

- 我们定义了一个callPackage函数，它的第一个参数是一个set，而另一参数则是一个函数f =>
    
    它用builtins.functionAttr函数得到f函数所需的参数并形成一个set =>
    
    它用builtins.intersectAttrs函数从我们给的第一个set里面取出f函数所需的参数并赋上前者的值 =>
    
    然后他执行f函数并用新形成的参数
    
- 我们完成了我们想要的 我们调用了一个函数并且给了一些可能的参数然后它会自动从里面选用它需要的参数
    

##### 假如我们所给的可能的set中缺少了函数所需的参数会怎样？

没什么特别的，报错而已~

- 所以其实我们传入这个set时可以给整个nixpkgs然后让函数从里面自己选

##### 增加override功能

```sh
callPackage = set: f: overrides: f ((builtins.intersectAttrs (builtins.functionArgs f) set) // overrides)
nix-repl> callPackage values add { }
8
nix-repl> callPackage values add { b = 12; }
15
```

- 我们新增了一个参数。这个参数是一个set。并会在生成最后的参数之后将其的值更新，形成真正最终的参数

### 将callPackage应用到我们的repository

```nix
let
  nixpkgs = import <nixpkgs> {};
  allPkgs = nixpkgs // pkgs;
  callPackage = path: overrides:
    let f = import path;
    in f ((builtins.intersectAttrs (builtins.functionArgs f) allPkgs) // overrides);
  pkgs = with nixpkgs; {
    mkDerivation = import ./autotools.nix nixpkgs;
    hello = callPackage ./hello.nix { };
    graphviz = callPackage ./graphviz.nix { };
    graphvizCore = callPackage ./graphviz.nix { gdSupport = false; };
  };
in pkgs
```

- nixpkgs这个变量为\<nixpkgs>里面所有的nix表达式，正如预期的要提供一个足够大的set
    
- callPackage接受两个参数一个是需要调用的函数的path,还有一个就是overrides的一些参数 =>
    
    它执行了path的表达式并向其默认提供了nixpkgs这个超大的集合让表达式自己挑选它需要的参数。如果需要自定义再通过默认来实现
    
- 然后构建了一个叫做pkgs的drv的集合
    

#### autotools.nix后传入的nixpkgs与我们生成的新callPackage函数的异同?

- 都是传入了一个的超大的全集一样的set
- 但是callpackge会选用刚好需要的



