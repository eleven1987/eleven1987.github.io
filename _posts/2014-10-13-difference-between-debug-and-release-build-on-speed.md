---
layout: post
title: ".NET 的 Debug 和 Release build 对执行速度的影响"
date: 2014-10-13 16:33:20 +0800
comments: true
categories: performance jit compiler
---

在真正开始讨论之前先定义一下 Scope。

* 本文讨论的范围限于执行速度，内存占用什么的不在评估的范围之内。
* 本文不讨论算法：编译器带来的优化基本上属于底层的优化，难以从质上提升执行速度。程序的快慢主要影响因素是采用的数据结构和算法这些高层次上的东西。我们接下来的讨论建立在这些高层次的东西已经被充分考虑的基础之上。

<!--more-->

## 目录

* .NET 的 Debug 和 Release build 对执行速度的影响
    * 如果你没有时间
    * Debug 和 Release build 的主要差异
    * 观察 JIT 优化的代码
    * JIT 优化对不同场景的影响
        * 迭代和内存操作
        * 非频繁的库调用
        * 频繁的库调用
        * 频繁的闭包调用
    * 结论

## 如果你没有时间

那么答案是：

* 对执行速度有影响。影响主要是由 JIT 而非 IL 编译器引入的。
* 一般来说 Release build 使得本程序集内的代码执行速度更快，而对第三方库的调用则几乎没有影响。
* 对于本程序集内代码来说，Release build 对迭代和内存操作加速效果比较明显。

## Debug 和 Release build 的主要差异

Debug 和 Release build 的一个最主要的区别是 Release build 会添加  [/optimize](http://blogs.msdn.com/b/ericlippert/archive/2009/06/11/what-does-the-optimize-switch-do.aspx) 选项。这个选项完成了两个任务：优化 IL 代码以及添加元数据。有意思的是第一项对性能提升的影响并不大。这是因为 CLR 应用的性能提升主要是由于 JIT 编译器而非具体的语言编译器完成的。语言编译器所完成的优化是很有限的，例如（不限于下面这些）：

* 当一个表达式的逻辑仅仅有一个作用而无其他副作用的时候将只会把产生这些副作用的代码进行生成；
* 忽略无用的赋值。例如 `int foo = 0`。因为我们知道 memory allocator 会将其初始化为 0（注：这是 IL 一级而非语言一级的）；
* 当静态类没有需要初始化的 field，或者 field 初始化为默认值时忽略对静态构造函数的生成；
* 忽略迭代中的局部未引用的变量（包括在仅仅在闭包的迭代中的未引用的外部变量）；
* 复用函数的栈空间（局部变量复用，删除未使用的局部变量）；
* 减少对局部变量（例如 `if` 和 `switch` 表达式的结果，以及函数调用的返回值）存储的要求而尽量的使用栈空间；
* 优化 branch 跳转指令；

这些优化都非常的直接，如果查看程序集的 IL 语句，会发现 /optimize 打开或关闭的情况下生成的代码几乎是相同的。不会有 IL 内联优化和循环的展开这种高级优化。因此性能提升不大。

__但是__，有一条 IL 优化对性能提升做出了相当的贡献，这里特别介绍一下：

* 删除了一部分为 breakpoints 定位以及 edit and continue 而插入的 nop 指令，若知详情如何，请看下面的 Tips。

真正起做用的是 /optimize 选项的第二个任务，更改 `DebuggableAttribute` 的参数。在添加 /optmize 的情况下该参数为 `IgnoreSymbolStoreSequencePoints`，而不会包含 `DisableOptimizations` 与 `EnableEditAndContinue`。这会实际的影响 JIT 编译器生成代码的策略。

> Tip:
> 
> MSDN 对 `IgnoreSymbolStoreSequencePoints` 解释为：使用 MSIL 的序列点而非 PDB 的序列点。JIT 编译器不会将两个序列点的进行合并优化编译，因而使用 PDB 文件中提供的序列点就可以保证编译结果和 PDB 严格对应从而提供更好的 Debug 体验。
>
> 有的同学就开始__激动__了，那么我如果使用 Debug Build 但是将 PDB 删除是否能够有和 optimize 一样的性能呢？显然不是的！由于加载 PDB 引发的这种性能问题在 .NET 2.0 的时候就已经解决了。解决的方式就是在 Debug build 中添加了 nop 指令作为隐式的序列点。从那之后，即使在 Debug build 下，其 `DebuggingModes` 也会包含 `IgnoreSymbolStoreSequencePoints` 选项了（因为根本没有必要加载 PDB）。
> 
> 此时应该明白了为何在 Release build 下删除了一些 nop 指令会使得执行效率得到提升，因为删除 nop 指令使得 JIT 编译器可以在相对大的范围内进行代码优化。
> 
> 但是如果入口应用程序是 Debug build 会不会影响 .NET BCL 或者第三方库的 JIT 编译结果呢？这是不会的，因为这个属性是 assembly scope 的，只要你使用的是 optimize 过的第三方库都会得到优化的 JIT 代码。

## 观察 JIT 优化的代码

本文不会广泛展示 JIT 优化的结果，但是如果你希望对比一下 Debug 和 Release build 下的 JIT 编译结果必须首先更改 Visual Studio 的默认 Debug 设置。

在 Visual Studio 中选择 Tools -> Options -> Debugging -> General。

* 取消 Enable Just My Code（这是因为优化的代码不属于 My Code 的范畴）
* 取消 Suppress JIT optimization on module load (Managed only)（防止在 Visual Studio 启动项目时阻止 JIT 优化）

至此就可以使用 Visual Studio 的调试器，在断点命中时通过 disassembly 窗口观看优化后的汇编代码了。

## JIT 优化对不同场景的影响

即便我们认识到开启 optimize 有可能使执行速度得到提升，但是在不同的使用场景下，其提升效果是不同的。

### 迭代和内存操作

场景之一是自己的代码中包含比较多的算法成分（并不是调用系统或者第三方库的算法而是自己实现算法）。算法中最典型的即极多的迭代操作和内存读写，因而我们选择插入排序作为测试算法。

```csharp
// sample code
int length = collection.Count;
for (int outerIndex = 0; outerIndex < length; ++outerIndex)
{
    int minimumIndex = outerIndex;
    T minimum = collection[outerIndex];
    for (int innerIndex = outerIndex + 1; 
        innerIndex < length; 
        ++innerIndex)
    {
        if (collection[innerIndex].CompareTo(minimum) >= 0) 
        {
            continue;
        }
        
        minimumIndex = innerIndex;
        minimum = collection[innerIndex];
    }

    Utility.Swap(collection, outerIndex, minimumIndex);
}
```

测试结果如下：

> __Iteration on value type test (selection sort on 20000 32-bit int array)__
> 
> * Debug build: 4.56s
> * Release build: 1.81s

我们必须确认两种不同的 build 的执行速度提升确实发生在迭代和内存读写上。通过 Profiling 我们可以证实这一猜想。其性能提升主要发生在循环体迭代，也就是 `for (int outerIndex = 0; outerIndex < length; ++outerIndex)`，数组数据读写，以及细小方法调用 `collection[innerIndex].CompareTo(minimum)` 上。其优化手法主要是尽量使用寄存器而不是内存寻址。

例如，内层循环 `for (int innerIndex = outerIndex + 1; innerIndex < length; ++innerIndex)` 在 Release build 下被编译为：

```
// outerIndex + 1
00007FFCBA5746F2  inc         ebx
// stack pointer change
00007FFCBA5746F4  inc         ebp 
// compare innerIndex to length
00007FFCBA5746F6  cmp         ebx,esi
00007FFCBA5746F8  jl          00007FFCBA5746A0
```

而 Debug build 是这样的

```
// read outerIndex to eax, increase eax then stores the value back
00007FFCBA594BA5  mov         eax,dword ptr [rbp+7Ch]  
00007FFCBA594BA8  inc         eax
00007FFCBA594BAA  mov         dword ptr [rbp+7Ch],eax
// set ecx to 0
00007FFCBA594BAD  xor         ecx,ecx  
// load length to eax
00007FFCBA594BAF  mov         eax,dword ptr [rbp+8Ch]  
// compare with increased outerIndex and to set the flag, move the flag value to eax and test if the value is true or not
00007FFCBA594BB5  cmp         dword ptr [rbp+7Ch],eax  
00007FFCBA594BB8  setl        cl  
00007FFCBA594BBB  mov         dword ptr [rbp+64h],ecx  
00007FFCBA594BBE  movzx       eax,byte ptr [rbp+64h]  
00007FFCBA594BC2  mov         byte ptr [rbp+77h],al  
00007FFCBA594BC5  movzx       eax,byte ptr [rbp+77h]  
00007FFCBA594BC9  test        eax,eax  
00007FFCBA594BCB  jne         00007FFCBA594B04  
```

JIT 还将 `int.CompareTo` 的调用进行了内联。在本例中，其贡献达到了 50% 左右，但是这个提升只在所有操作都基本是细小操作的时候才会显现。

从上述分析中不难看出，/optimize 对迭代中的内存操作的优化非常有效，因此如果我们迭代的并非 value type 而是需要多次进行寻址（因为要不断的使用其 field 值）的 reference type 则性能提升也会非常明显。

> __Iteration on reference type test (selection sort on 20000 ref instance array. The ref type contains 1 int field)__
> 
> * Debug build: 11.57s
> * Release build: 4.00s

类似的操作还例如 DTO 之间的映射，这个操作也属于迭代式的内存密集形操作。在如下的测试代码：

```csharp
var source = Enumerable.Range(0, dataAmount)
    .Select(
        i => new Dto
        {
            Name = new NameDto
            {
                FirstName = "firstname" + i,
                Middle = "Q",
                LastName = "lastname" + i
            },
            Age = 20 + i % 10
        });

var destination = source.Select(
    e => new
    {
        FirstName = e.Name.FirstName,
        MiddleName = e.Name.Middle,
        LastName = e.Name.LastName,
        Age = e.Age
    });

m_count = destination.Count();
```

每一次 5,000,000 个迭代测试的情况下能够获得 15% 以上的执行速度提升。其主要的优化手段仍然是尽量的使用寄存器。

### 非频繁的库调用

该场景下仅对系统或者第三方库进行非频繁调用。非频繁的调用有两种情况，第一种情况属于调用的方法仍是有相当复杂程度的算法，这是非频繁调用的常见情况；第二种是非频繁调用的方法也非常简单，但是这对性能影响不大因此我们只关注第一种情况。

在测试之前，不妨预测一下，由于我们系统 BCL 和第三方库均使用 /optimize 进行 build，因此对于非频繁的库调用，我们的代码优化的空间并不大，性能数据应当非常接近。以下是测试结果。

> __Infrequent lib calls (quick sort 9,000,000 integers x 5 runs)__
> 
> * Debug build: 6.37s
> * Release build: 6.62s

### 频繁的库调用

频繁的库调用往往包含对细小的操作进行的调用。我们着重关注 Parsing 和 ToString 这两种常见的操作。这是因为在 Web App 中，Serialize - deserialize 是最频繁而常见的操作。

同样我们可以预测执行的结果。由于是频繁操作，因此迭代部分的性能会有一些增强。但是相比于迭代，库调用的时间要长的多，因此这种性能增益几乎是不可见的。可以预见其性能数据应当是非常接近的。

测试代码范例：

```csharp
double next = 1d + random.NextDouble();
total += double.Parse(next.ToString(CultureInfo.InvariantCulture));
```

> __Frequent lib calls (serialize/deserialize `double` x 5,000,000 times)__
>
> * Debug build: 6.89s
> * Release build: 6.77s

为了保证测试的有效性我们仍然需要确认性能的消耗主要发生在 serialize - deserialize 上。Profile 结果和我们预想是一致的：

```csharp
for (int i = 0; i < iterationCount; ++i) // 0.5%
{
    double next = 1d + random.NextDouble(); // 1.3%
    total += double.Parse(
        next.ToString(CultureInfo.InvariantCulture)); // 98.2%
}
```

### 频繁的闭包调用

我们关注频繁的闭包调用，因为 LINQ 以及事件处理已经得到了非常广泛的应用。其典型形式是使用匿名函数或 lambda 表达式作为回调方法。回调方法往往执行数据的加工（Select）或者筛选（Where）。

由于使用 LINQ 就是库调用，因此迭代的优化不论 Debug 还是 Release build 都会发生，唯一的优化空间只是匿名委托的内联以及寄存器的使用，但这样也不会带来什么性能提升，因为大多数情况下匿名函数的执行时间要比 call 长的多。可以预见，Debug build 和 Release build 的性能指标是比较接近的。

测试代码范例：

```csharp
IEnumerable<double> enumerable = Enumerable
    .Range(1, iterationAmount)
    .Select(
        i =>
        {
            string str = i.ToString(CultureInfo.InvariantCulture);
            int operand = int.Parse(str);
            return Math.Pow(operand, factor);
        })
    .Where(i => i > 0.2);
double total = enumerable.Average();
```

> __Frequent closure calls (for 8,000,000 iterations)__
> 
> * Debug build: 4.51s
> * Release build: 4.47s

## 结论

可见 JIT 编译器对 BCL 以及 Release build 下的第三方库调用影响并不大，因为本地代码本身并不占有很多的比重，典型的情形例如数据库查询。但是对于本地代码占有很高比重，且其中包含大量的迭代和内存操作的情形（光线追踪，服务端页面生成（非预编译的情形），批量 DTO / Entity 映射）的可以起到比较不错的优化效果。

因此，从执行速度的角度上考虑，推荐在 Package/Deployment 的时候切换至 Release build。