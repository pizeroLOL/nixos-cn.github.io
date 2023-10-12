# 若干个函数凝聚成一个巨大的函数

## 为什么 NixOS 采用函数式的设计

NixOS 在设计之初就竭力遵照函数式的设计，函数式有什么迷人的气质吗？

- NixOS 需要保证构建结果的**可复现性**：为了实现同样的输入能有同样的输出，采用纯函数式作为设计范式可以避免构建过程被无关变量干扰或不改变环境（无副作用），保持构建结果的统一也保证了可预测性。

- **完整依赖性**：当我们要构建一个包时，Nix 会首先检查这个函数的所有输入（即依赖项）是否都已经存在。如果所有的输入都存在，Nix 就会执行这个函数（即进行构建）。如果有任何一个输入不存在，Nix 就会先去构建那个输入。这就确保了每个包的构建过程都是完全独立的，只依赖于它自己的输入。这也意味着，如果我们改变了任何一个包的输入（例如更新了源代码或者更换了依赖项），Nix 就会重新构建这个包，这个包变动会让其他依赖它的包也会被重新构建。如果你的配置文件描述的依赖不完整，则无法进入下次构建。

- **原子性升级或回滚**：在函数式编程中，数据是不可变的。由于包管理操作从不覆盖 `/nix/store/` 中的包，而只是在不同的路径中添加新版本，因此该操作是原子性的。所以在包升级期间，不存在包有一些文件来自旧版本、一些文件来自新版本的时间窗口。

- **多版本并存**：每个包都是纯函数的输出。即使是同一软件包，只要它们的构建过程不是完全一致的，就会存储在其自己的哈希目录中，因此可以在系统中同时存在多个版本的同一软件包。这就像在函数式编程中，我们可以引用同一个函数的不同版本。

所以 NixOS 其实是一个庞大的构建系统，以用户的配置文件作为函数，Nix 则是这个函数的解释器或执行器，而函数的输出则是一个完全配置好的，可以直接使用的系统（代）。

所以，函数式的设计不只是单单体现在这个构建系统，还体现在它的打包语言 Nix 上面。

## Nix 语言的语法设计

Nix语言是一种纯函数式语言，其语法设计充分体现了函数式编程的特点：

- **不可变性**：在Nix中，变量一旦被赋值，就不能更改。

- **纯函数**：Nix 语言中的函数是纯函数，也就是说，给定相同的输入，它们总是会产生相同的输出，并且没有副作用。为了不破坏函数的纯净，Nix 语言没有全局变量的设计。

- **惰性求值**：Nix 语言使用惰性求值，这意味着表达式在需要时才会被求值。可以降低开销。

- **高阶函数**：Nix 语言支持高阶函数，也就是说，函数可以接受其他函数作为参数，或者返回函数作为结果，也可以产生不同状态的闭包。

::: note 闭包是什么
如果你没有接触过任何一门编程语言，你可以将闭包粗略地理解为带状态的函数。
:::

## 每一个 Nix 文件就是一个函数

Nix 源码文件是一种以 `nix` 为后缀的文本文件，让我们一览：

```nix
{ pkgs }:

pkgs.stdenv.mkDerivation {
  name = "my-package";
  src = ./source;
  buildInputs = [ pkgs.gcc ];
}
```

几乎所有的 Nix 源码文件都是这种格式：冒号前的是输入，冒号后的是输出。整个文件是被一个大函数所包裹住的，每个 Nix 文件就是一个函数。

## 一切皆函数

NixOS 本身就是一个超大号的函数，这个函数由它内部的若干函数驱动。