---
layout: post
title: "你真的非用 dynamic 不行"
date: 2015-02-26 15:38:51 +0800
comments: true
categories: csharp dynamic couple
---

自从 .NET 4.0 开始引入 `dynamic` 之后，好像将 C# 一夜之间变成了动态语言。使用场景不计其数。但是，你真的一定要用 `dynamic` 吗？

<!--more-->

## 微软推荐场景

首先介绍一下微软的推荐场景吧：Interop。由于动态语言运行时（DLR）的引入，Interop 已经从 Win32 API 和 COM 扩展到了与动态语言的交互。考虑到动态语言的 Schema less 的特性，的确是一个适用 `dynamic` 的场景。

```csharp
ScriptEngine engine = Python.CreateEngine();
ScriptScope scope = null;

scope = engine.CreateScope();
ScriptSource source = engine.CreateScriptSourceFromFile("Script.py");

source.Execute(scope);

dynamic mc = scope.GetVariable("ReturnObject")
```

而另一个场景就不这么直观了： COM Interop。以 Office COM API 为例。由于 Office COM API 的 Query 参数一般是 `IDispatch **`。这样在 C# 中的类型就全部变成了 `object`。因此我们必须首先将其转换为正确的接口类型再进行使用。援引微软官方的例子：

```csharp
((Excel.Range)workSheet.Columns[1]).AutoFit();
((Excel.Range)workSheet.Columns[2]).AutoFit();
```

而对于 Visual Studio 2010 之后的 Interop Assembly，将 `object` 转换为了 `dynamic`。 因此我们只需要这样写就行了：

```csharp
workSheet.Columns[1].AutoFit();
workSheet.Columns[2].AutoFit();
```

好的，优雅多了，少了很多类型转换的工作。__但是__如果你不小心打错了一个字的话就只能指望运行时再去慢慢发现了。另外，这也意味着某些代码分析功能，例如 _find usage_ 变得不可用或者不准确。如果不用 `dynamic` 呢？只需要很小的努力就可以避免这些问题。

```csharp
public static Excel.Range GetColumns(
    this Excel.WorkSheet workSheet, 
    int index) 
{
    return (Excel.Range)workSheet.Columns(index);
}

workSheet.GetColumns(1).AutoFit();
workSheet.GetColumns(2).AutoFit();
```

个人觉得这种小小的努力在保持了目标程序优雅程度的同时，比起需要不断的翻阅文档确认某个对象是否具备某些属性或者方法的维护成本要低一些。不过考虑到 Office API 的庞杂，如果你的确较为广泛的使用了这些 API，这样自行封装的成本不能再忽略的时候使用 `dynamic` 也可以是一种选择。

综上，对于和原生的动态语言交互，`dynamic` 的确非常适用；而对于 COM Interop 来说，`dynamic` 并非是一个非常适用的场景。

## 静态与动态

话分两头，在讨论到底用谁之前，先来关注一下耦合。

### 物理耦合和逻辑耦合

和物理世界一样。在足够优化的情况下，耦合也维持着守恒。耦合分为两种，一种是程序之间显式的使用，包含，聚合等关系。这种关系是可见的，静态可分析的，称之为 __物理耦合__。另外一种耦合关系则不同，具有耦合关系的两个部分并没有可见的，静态的关联。而是隐含在各自的程序之中。当一个部分发生变化的时候，另一个部分的行为也将被影响。称之为 __逻辑耦合__。

> 假设有两个模块，一个模块负责加密，另一个模块负责解密。两个模块不共用任何代码各自独立实现。

> 一开始，他们各自正确实现了 DES 算法的加解密。这样，通过加密模块生成的密文能够正确的被解密模块解密。但是，由于 DES 算法的安全性较差，加密模块更改了自己的算法。这时，解密模块便不能够正确的解析加密模块生成的密文了。

> 上述例子中，加密解密模块虽然不存在任何形式的物理耦合，但是却存在逻辑耦合。在加密模块改变自己的算法的时候，没有任何静态分析的办法得知到底有多少模块会为此而被影响。

逻辑耦合有三种别名：

* 在逻辑耦合被广泛接受的情况下，称之为 _Convension_；
* 在逻辑耦合被良好的定义和记录的情况下，称之为 _Contract_；
* 在并不满足上述两种情况时，逻辑耦合往往只停留在一两个人的脑子里，称之为 __烂代码__。

### 耦合的守恒与静动态的选择

虽然物理耦合和逻辑耦合和静态类型系统和动态类型系统并没有严格的对应关系，但却能代表其典型的应用场景。例如，在静态类型系统中显式定义的使用，包含，聚合等关系是物理耦合的典型表现形式。而 `dynamic` 则属于逻辑耦合的形式。

当选择了静态类型系统时，大部分耦合关系表现为物理耦合的形式，各个部分关系相对显式而清晰。即使程序具备复杂的结构，其知识的传递和__维护__也相对容易。若选择了动态系统，则可以获得更加优雅灵活的代码，但由于耦合是守恒的，因此必须用额外的形式（例如测试，文档等）对逻辑耦合进行记录以保证程序的可__维护__性。

因此，到底选择使用静态类型系统，还是选择使用 `dynamic` 应当取决于你要解决的问题。而不管采用哪种形式都必须要保证程序的可维护性。

> 即便是在动态类型语言中，我们也倾向于从写法上模拟静态类型定义。例如我们可能更倾向于

```javascript
var MyType = (function () {
    function MyType() {
        this._name = undefined;
        this._birthyear = undefined;
    }

    // ...

    MyType.prototype.set_name = function (name) {
        this._name = name;
    }

    return MyType;
})();
```

> 否则只有当看到了 `set_name` 的时候，才能够确定 MyType 具有 `_name` 这个 field。而如果为了避免使用 field 而将其全部暴露为属性，则又破坏了封装。

## 我们需要 `dynamic` 的地方并不多

### Contract 往往不需要 `dynamic`

`dynamic` 相关的代码在多数情况下都会引入逻辑耦合。因此我们可以考察逻辑耦合的三种形式：

* 我们很少有机会自己写 Framework。因此引入 Convension 的机会并不多。
* 谁都不希望自己写烂代码；
* 那么就剩下 _Contract_ 了。

假设有两个模块：_A_、 _B_。对于一个请求过程，_A_ 的请求对象为 _RequestA_，而 _B_ 的接收请求对象为 _RequestB_。则 _RequestA_ 和 _RequestB_ 的关系应该如下图所示：

<img src="{{root_url}}/images/blog/contract_relationship.png" alt="contract relationship"/>

从之前的讨论可知，我们关注的是 Schema 的部分而非业务领域的部分。则对于 _A_ 系统，从 Schema 的角度需要确保返回结构为 _ReqA' _ + _Intersaction_ 的对象而对于 _B_ 系统，需要确保可以接受 _ReqB' + Intersaction_ 或者 _Intersaction_ 的对象。即一共需要保证 3 种不同的情况。

如果使用静态类型系统，则需要定义 2 个 Dto

* _RequestA_ = _ReqA'_ + _Intersaction_
* _RequestB_ = _ReqB'_ + _Intersaction_

另外，还需要做一种情形的 Contract 测试或者补充文档说明：即 _Intersaction_ 部分的测试/文档。

若使用 `dynamic` 来代表 DTO。则需要在系统 _A_ 中添加对 Response 的测试/文档；需要在系统 _B_ 中添加对两种情况的 Contract 测试/文档。但是，即使添加了这些测试/文档，也很难搞清系统 _A_ 和系统 _B_ 的 Contract 的全貌。而在实际中，大部分项目仅仅添加了 _Intersaction_ 部分的测试。随着系统 _A_ 和系统 _B_ 不断独立的发生变化，其 Contract 的 Schema 将变得更加难以维护。

例如，当你看到这样的 API 的时候，你觉得应该怎么做才能搞清楚 Request 的 Model 里面有什么呢？

```csharp
public HttpResponseMessage Upload([FromBody] dynamic fileUploadRequest) {
    // ...
}
```

而这样的时候，是否能够非常容易的搞清楚呢？并且还不用写那么多测试/文档。

```csharp
public HttpResponseMessage Upload([FromBody] FileUploadRequest fileUploadRequest) {
    // ...
}
```

### 局部上下文中使用匿名对象代替 `dynamic`

在封装的局部上下文中，有时需要引入一些临时的数据结构。此时，可以考虑使用匿名对象代替 `dynamic`，即倾向于使用：

```csharp
var temporaryStructure = new {
    Name = new {
        FirstName = "Tom",
        LastName = "Hall"
    },
    BirthYear = 1985
};
```

而不是使用

```csharp
dynamic name = new ExpandoObject();
name.FirstName = "Tom";
name.LastName = "Hall";
dynamic temporaryStructure = new ExpandoObject();
temporaryStructure.BirthYear = 1985;
```

## 综合

我们没有办法消灭程序的必要信息，对于耦合，要么是物理耦合，要么是逻辑耦合；对于Schema，要么定义在程序中，要么存在在你的思维里。而让程序变得可维护的方法，就是尽量用程序（定义静态类型，或者写测试）来表示这些信息。因此，综合考虑一句话总结：能不用，就别用。

那么什么时候可以考虑使用呢？

* 当我们希望和动态语言引擎进行 Interop 的时候；
* 当我们书写 Framework 并期望为其引入某些 Convension 的时候；

除了上述两种情况之外，还存在一种额外的情形。不过这篇文章就不展开了。

* 当传统的继承组合体系不能满足要求的时候（例如，有非常强烈的使用 mixin 的要求）。