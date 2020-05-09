# 【Crash Course】Just-In-Time (JIT) Compilers

## Introduction

[【Crash Course】Compiler & Interpreter](https://github.com/suukii/Articles/blob/master/articles/%E3%80%90Crash%20Course%E3%80%91Compiler%26Interpreter.md) 这篇介绍了 Compiler 和 Interpreter 以及它们各自的优缺点，我们先来简单回顾一下。

- Compiler: 事先把源码编译成机器语言，在需要执行程序的时候直接运行编译后的代码。因为编译工作是事先完成的，所以 Compiler 在编译的时候有充足的时间好好分析源码，并对其进行优化，生成运行速度更快的代码。

- Interpreter：在需要执行程序的时候才开始边编译，边运行编译后的代码。如果是在运行一个循环语句，那代码每次循环都要重新编译一次。而且由于是一边编译一边运行，时间很紧，Interpreter 也不能在编译代码的过程中做太多优化的工作，生成的代码运行速度相对慢一些。

而 JIT 则是结合了 Compiler 和 Interpreter 的优点，是两者的一个完美结合。

## How JIT Works

- 在程序刚开始运行的时候，JIT 像 Interpreter 一样一行一行地编译并运行代码；

- 在这个过程中，如果它发现了一些代码需要重复地运行(比如循环中的代码)，就会像 Compiler 一样把这部分代码进行编译并存起来，下次再需要运行这部分代码时，就从内存中取出编译后的代码直接运行；

## How JIT Compiles Codes

如果 JIT 发现了需要多次运行的代码，它会把这部分代码进行编译，但编译的方式分两种，分别由 Baseline Compiler 和 Optimizing Compiler 来负责。

### Baseline Compiler

Baseline Compiler 负责做一些简单的编译工作，编译时间不长，但生成的代码优化程度也不高。如果把代码优化的过程比喻成论文修改的话，Baseline Compiler 就只是改改标点符号和错别字而已。

### Optimizing Compiler

Optimizing Compiler 负责将源代码编译成优化程度更高的代码，编译时间较长，不过生成的代码运行速度更快，相当于对论文的结构和内容进行了优化。

### How JIT Knows Which To Use

那 JIT 怎么知道一行代码是否需要优化？

在代码执行的时候，JS 引擎会用一个 `monitor`(aka. `profiler`) 来监测并记录代码的执行频率。

- 如果一行代码被重复执行了几次，`monitor` 会将它标记为 `warm`，JIT 就会把它送到 Baseline Compiler 去编译，然后把编译后的代码存起来。

- 如果一行代码被重复执行了很多很多次，它就会被标记为 `hot`，JIT 就会把它送到 Optimizing Compiler 去编译，然后把编译后的代码存起来。


## Extension

通过监测代码执行频率以及编译优化代码，JIT 提高了 JS 的运行速度，不过它同时也带来了一些 overhead:

### 优化和反优化(optimization and deoptimization)

上文没有提到的是，Compiler 在优化代码的时候会作出一些假设。

- 比如说函数 `function foo(a) {}` 由于执行较频繁被送到了 Optimizing Compiler 去优化；

- 前提：`foo` 在最初几次执行的时候接收到的参数 `a` 都是字符串类型的；

- Compiler 会假设在之后的调用中，传入的 `a` 也都是字符串，并基于这个假设对 `foo` 进行优化；

- 如果程序之后调用 `foo` 时都是传入字符串，那一切就都很美好；

- 不过，假如有一次调用中传入了数字，因为对数字和字符串的操作不一样，之前基于假设 `a 是字符串` 优化的代码就完全用不上了，这时 Compiler 就会将代码反优化并重新进行编译和优化；

- 假如在程序运行中 Compiler 做了很多这样错误的假设，一直在优化和反优化同一段代码的话，有可能消耗的时间比运行没有优化的代码还长；

- 不过，浏览器对此也做了一些处理，如果优化和反优化这个过程重复了太多次的话，就直接放弃了对这段代码进行优化。

### 额外的内存占用

- monitor 记录的代码执行频率等信息需要占用一部分内存

- 编译后的代码也需要地方储存起来
