---
title: nixLanguage
publish: true
---

## nix language

### data

#### 字符串的表达

##### 字符串有三种表达方式

“” ’’ ’’ opening’‘后接的空格将会被忽略 url可以不用加 “”

#### 路径path

##### 在nix中path被视为一种单独的类型，它的特点是必须要包括斜杠/

##### path也可以用<>包裹，其含义是将在NIX\_PATH这个环境变量中搜寻被包裹的文件或者目录名称

- 🔗To -> [&lt;nixpkgs&gt;这个块代表什么意思？](http://localhost:1313/posts/nixrepl/#6485785e-2721-450b-8fc3-e984498d6bf1)

##### 当interpolated string为路径时，会将其先拷贝到nix store下然后通过返回其store下的路径得到结果

 >For instance, evaluating “${./foo.txt}” will cause foo.txt in the current directory to be copied into the Nix store and result in the string “/nix/store/hash-foo.txt”.




##### 除了<>包括的PATH不能用interpolated string其余均可，但是同样的必须在其前有/斜杠

> At least one slash (/) must appear before any interpolated expression for the result to be recognized as a path.
> a.\\({foo}/b.\){bar} is a syntactically valid division operation. ./a.\\({foo}/b.\){bar} is a path.

*********

#### boolean

##### true 与 false 为布尔值的表示而非0 1之类的

#### list

##### list如何表示？

###### 用[]

##### list中的item如何间隔

###### 用空格

#### attribute set

##### 什么是attribute set?

###### 即键值对的集合

##### 如何表示attribute set?

###### 用{key=value;}

##### attribute set中的item如何隔开？

###### 用; 注意与list中的空格相区别

##### 如何读取set中的item值？

###### 演示

```nix
{ a = "Foo"; b = "Bar"; }.a
```

##### 如何读取的值不存在如何设置默认值？

###### 演示

```nix
{ a = "Foo"; b = "Bar"; }.c or "Xyzzy"
# 用or逻辑操作嘛
```

##### 在对set进行操作时变量名字符串替换如何实现？

###### 演示

```nix
let bar = "bar"; in
{ "foo ${bar}" = 123; }."foo ${bar}"
```

##### 当键值对的key为null类型时的后果是？

###### 会报error

##### 键值对拥有\_\_functor键时的作用是什么？

###### 演示

```nix
let add = { __functor = self: x: x + self.x; };
    inc = add // { x = 1; };
in inc 1
```

evaluate to 2

###### 在键值对中//双斜杠的含义是什么？

- 用右侧键值对

###### 如何理解这个\_\_functor,其将其本身的set作为第一个参数传给解释器 ，然后再接受第二个参数 在 上述中的self: x: x+self.x 中的x并非同一x，第一次出现的x是第二个参数，第二次出现的x是第二个参数too，第三次处现的self.x中的x则是作为set本身的key value中的x key

```nix
nix-repl> let add = { __functor = self: y: y + self.x; };
          inc  = add // {y=1;};
          in  inc 2
error: attribute 'x' missing

       at «string»:1:38:

            1| let add = { __functor = self: y: y + self.x; };
             |                                      ^
            2| inc  = add // {y=1;};
       Did you mean y?
```

例如此处我们将提供的set本身中的键改成y则其就会报错

###### 我们如何调用functor并为其提供其 自身？

- 用//操作符号更新set，所 更新的后者就是其本身
    
    *********
    

### 语言结构

#### recursive sets

##### 什么是及如何表达recursive sets

###### 如上所述sets的表达是{key=value;} 而recursive sets即在 其前放加上rec关键词

##### recursive sets其作用是什么？

###### 在普通的set中键值对不能互相引用其间的值

###### 而在recursive sets中使得键值对可以引用其他键值对的值比如

```nix
rec {
  x = y;
  y = 123;
}.x
```

#### let 表达式

##### let表达式的写法是？

###### let x=“foo”; y=“bar”; in x+y

##### let表达式的作用是？

###### 就是创建局部变量并 赋值以用于in关键字后面的函数

#### [Inheriting attributes](https://nixos.org/manual/nix/stable/language/constructs.html#inheriting-attributes)

#### [](https://nixos.org/manual/nix/stable/language/constructs.html#inheriting-attributes)

- 🔗To -> [需要再次注意inherit关键词的用处是仅限于？](http://localhost:1313/posts/nixderivation%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/#6485c8c5-d909-4576-ab2c-1e8a1b7bf63a)

##### inheriting attributes的作用是什么？

###### 从外围的局部变量中继承它的变量名，同时也继承它的值

###### 比如我用let x \= foo; in {x \= x} 也可以写作 let x=foo; in {inherit x;}

##### 如何继承外围环境set中的变量值呢？

###### 演示

```nix
...
inherit x y z;
inherit (src-set) a b c;
...

...
x = x; y = y; z = z;
a = src-set.a; b = src-set.b; c = src-set.c;
...
```

###### eg:

```nix
nix-repl> let set={a=1;b=2;c=3;}; in { inherit (set) a b;}.a
1
```

注意在inherit时其后接的item不用跟,或这;等其他符合 而是用空格隔开

#### function

##### 参数和函数体如何写？

###### 参数：函数体;

##### 如何传递多个参数？

###### 参数1:参数2:参数3:函数体;

有点嵌套的感觉，第一个参数返还了包含另一个参数的函数 类推

###### 或者用set pattern {a,b,c}作为一个参数然后传abc三个变量

- 需要注意的是set pattern的写法中分隔变量的是逗号，而set的完整写法是{a=foo;b=bar;c=lalala;}用引号分隔。
    
    ```nix
    nix-repl> a = {x,y,z}: x+y+z
    
    nix-repl> a {x=1;y=2;z=3;}
    6
    ```
    

- 用set pattern作为参数时中的变量必须是你传参时全部包含所拥有的，如果你所传的set包含了patter之外的则不被允许
    
    ```nix
    nix-repl> a {x=1;y=2;z=3;m=5;}
    error: anonymous function at «string»:1:2 called with unexpected argument 'm'
    
           at «string»:1:1:
    
                1| a {x=1;y=2;z=3;m=5;}
                 | ^
           Did you mean one of x, y or z?
    ```
    

- 用set作参数时如何才能传递未被函数pattern所包含的变量？
    
    - 用…省略号
        
        ```nix
        { x, y, z, ... }: z + y + x
        
        nix-repl> a = {x,y,z,...}: x+y+z
        
        nix-repl> a {x=1;y=2;z=3;m=5;}
        6
        ```
        

- 如果在函数中引入3个变量时际只使用两个变量而我只传入两个变量会发生什么？
    
    - 演示
        
        ```nix
        nix-repl> a = {x,y,z}: x+y
        
        nix-repl> a {x=1;y=2;z=3;}
        3
        
        nix-repl> a = {x,y,z}: x+y+z+m
        error: undefined variable 'm'
        
               at «string»:1:17:
        
                    1|  {x,y,z}: x+y+z+m
                     |                 ^
        ```
        
        也就是传入更多使用更少是允许的，传入少使用多是不被允许的
        

- 如果在函数中引入3个变量并且都使用而我只传入2个变量会发生什么？
    
    - 演示
        
        ```nix
        nix-repl> a = {x,y,z}: x+y+z
        nix-repl> a {x=1;y=2;}
        error: anonymous function at «string»:1:2 called without required argument 'z'
        
               at «string»:1:1:
        
                    1| a {x=1;y=2;}
                     | ^
        ```
        
    
    - 如何给参数设置默认值以解决此种情况？
        
        - 演示
            
            ```nix
            nix-repl> a = {x,y,z ? 1}: x+y+z
            
            nix-repl> a {x=1;y=2;}
            4
            ```
            

- @ patter的意义是什么？
    
    - 给传递的set赋予一个变量名使得调用额外的参数称为可能
        
        如上所述有些函数的参数是写死数量的，但是有些是可变的存在一些可选的参数，那么我们如何在函数体中使用哪些可选的参数呢？…省略号允许我们传递超过所需数量的参数这使得optional变为可能，同样我们可以用@符号来使得我们能让我们传递的set赋给一变量从而可以使用.符号来调用其中整个的key value对
        
        ```nix
        { x, y, z, ... } @ args: z + y + x + args.a
        # the same as { x, y, z, ... } @ args: z + y + x + args.a
        ```
        
        *************
        



#### 条件判断

##### 条件判断的语法是？

###### if foo then bar else lala

###### foo为判断的布尔值，而bar 和lala则为true或false时相应执行的语句

*********

#### assert

##### assert语法是?

###### assert foo; bar

###### 如果foo 是true则执行bar

###### 演示

```nix
{ localServer ? false
, httpServer ? false
, sslSupport ? false
, pythonBindings ? false
, javaSwigBindings ? false
, javahlBindings ? false
, stdenv, fetchurl
, openssl ? null, httpd ? null, db4 ? null, expat, swig ? null, j2sdk ? null
}:

assert localServer -> db4 != null; ①
assert httpServer -> httpd != null && httpd.expat == expat; ②
assert sslSupport -> openssl != null && (httpServer -> httpd.openssl == openssl); ③
assert pythonBindings -> swig != null && swig.pythonSupport;
assert javaSwigBindings -> swig != null && swig.javaSupport;
assert javahlBindings -> j2sdk != null;

stdenv.mkDerivation {
  name = "subversion-1.1.1";
  ...
  openssl = if sslSupport then openssl else null; ④
  ...
}
```

#### with 表达式

##### with表达试的语法是？

###### with e1; e2

##### with表达式的作用是？

###### 将e1这个set 代入表达式e2的作用域中

```nix
let as = { x = "foo"; y = "bar"; };
in with as; x + y

with (import ./definitions.nix); ...
#makes all attributes defined in the file definitions.nix available as if they were defined locally in a let-expression.
```

###### 演示

```nix
nix-repl> let a = 1; in let a = 2; in let a = 3; in let a = 4; in a
4

nix-repl> let a = 3; in with { a = 1; }; let a = 4; in with { a = 2; };a
4
```

###### 可以理解成相当于是进入到了set中可以直接调用其中的值或者，说把 set中的每一项都用let声明在了新环境中

#### [string interpolation](https://nixos.org/manual/nix/stable/language/string-interpolation.html#string-interpolation)

##### 什么是string interpolation?

###### 就是在string中用变量的方法

###### 比如"I ${foo} you" 当我在外部用let 定义 foo=love或者hate时 string也变成了I love you 或者I hate you


### operator