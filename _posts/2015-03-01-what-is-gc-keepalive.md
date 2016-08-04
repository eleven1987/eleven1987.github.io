---
layout: post
title: "有关 GC.KeepAlive 文章的吐槽"
date: 2015-03-01 01:38:51 +0800
comments: true
categories: clr csharp gc
---

之所以会写这个是因为今天无意之间看到了一篇文章：[C# and GC.KeepAlive()](http://manski.net/2013/08/c-and-gc-keepalive/)。这篇文章貌似正确事实上的论述是不严谨的。这对于吐槽星人来说无法容忍，故而喷一篇如下。

<!--more-->

## 回收对象而非回收引用

**原文**说：

> 垃圾收集器可以在**变量**最后一次使用后直接将其收集。(The .NET garbage collector may collect a variable directly after its last use)

这种论述是错误的。垃圾收集器 Collect 的是对象，和他的引用是没有关系的。

## Finalizer 不应该这样写

对于如下的程序：

```csharp
class SomeClass
{
    // This field is initialized somewhere 
    // in the constructor (not shown here).
    public SomeOtherClass Value;
     
    ...
}
 
...
 
void MyMethod()
{
    SomeClass obj = new SomeClass();
    SomeOtherMethod(obj.Value);
    YetAnotherMethod();
    // obj still alive here? Possibly not.
}
```

**原文**说：

> 通常情况下，这种代码不会造成任何问题。而当 `SomeClass` 具有 *Finalizer* 的时候情况就不一样了。如果 `obj` 的 *Finalizer* 在 `SomeOtherMethod()` 之前被调用，那么 `SomeOtherMethod` 将会因无法使用 `obj.Value` 而失败。(Usually that’s not a problem. It becomes a problem, however, if SomeClass has a finalizer. So when obj‘s finalizer is called already before SomeOtherMethod() is called, SomeOtherMethod() will fail because it can no longer use obj.Value.)

这种说法有一半是**错误**的。这句话里有两个命题：

* 第一：`obj` 所引用的对象可能在 `SomeOtherMethod()` 调用之前被回收；
* 第二：如果 `obj` 所引用的对象被回收了，并且 `obj` 具有 *finalizer*，那么 `obj` 的 *finalizer* 将使 `obj.Value` 不可用。

第一个命题是正确的。具有和 Root 拥有强引用链接路径的对象不会被回收。所谓 Root 指的是：

* 寄存器
* 栈
* global 和 static field

因此在 `obj.Value` 使用之后，再没有使用 obj 的场合，在 GC 的 *look-ahead optimization* 启动的情况下，可以回收 `obj`。而 `obj.Value` 由于仍然具备强引用而不会被回收。

第二个命题则是**错误**的。因为 *finalizer* 不应当使用 `Value` 成员！finalizer 只能够回收本实例的非托管资源而绝不可以触碰托管资源，在这个例子里面，SomeOtherClass 显然是一个托管资源，因此在 finalizer 里面对其进行的任何操作（包括资源释放）都是不可取的。

## GC.KeepAlive() 只是在规避 look-ahead Optimization

**原文**说：

> 使用 `GC.KeepAlive()` 就像是使用 `GCHandle` 一样，只是更加轻量级速度更快。(Using GC.KeepAlive() is like using GCHandle, just more light-weight and faster.)

事实上，`GC.KeepAlive()` 的实现是空的：

```csharp
[MethodImpl(MethodImplOptions.NoInlining)]
public static void KeepAlive(object obj)
{
}
```

这个函数仅仅是在阻止 GC 使用 look-ahead optimization 回收指定的 reference 所引用的对象。而 `GCHandle` 的行为却不止于此，尤其是在使用了 `Pinned` 选项的时候，GC 不但不能够回收，而且也不能够进行内存压缩（Compact），因而还必须在使用结束后通过 `GCHandle.Free()` 方法取消该固定指针。使用起来必须非常小心否则可能造成严重的性能问题。请不要这么轻描淡写的说他们就是一回事儿啊。

## 正确的说法

对于程序：

```csharp
void MyMethod()
{
    SomeClass obj = new SomeClass();
    SomeOtherMethod(obj.Value);
    YetAnotherMethod();
    // GC.KeepAlive(obj);
}
```

`obj` 所引用的对象可能在 `obj.Value` 被获取之后回收（由于 look-ahead optimization）。而通过使用 `GC.KeepAlive(obj)`，可以规避 look-ahead optimization，而确保 `obj` 所引用的对象在 obj.Value 和 GC.KeepAlive(obj) 之间都是有效的。

对于纯托管程序，不会涉及这种状况。这种情况一般出现在托管程序已经不再使用某非托管资源，而由于该非托管资源还在被非托管进程使用从而必须保证它不会被 GC 回收。